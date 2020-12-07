
# Bosh Deploy 

## 1. Bosh Deploy Error 해결 방법

```
$ bosh -d nginx deploy manifest.yml
Task 610 | 23:45:48 | Preparing deployment: Preparing deployment (00:00:02)
Task 610 | 23:45:50 | Preparing deployment: Rendering templates (00:00:00)
Task 610 | 23:45:50 | Preparing package compilation: Finding packages to compile (00:00:00)
Task 610 | 23:45:50 | Compiling packages: nginx/66c3f00b386cb6166d60748308b27fa31f4285ad2eac86024e7166b29837bf0d (00:01:46)
Task 610 | 23:47:51 | Creating missing vms: nginx/ca6c2391-685d-4e65-a3eb-2db32b436503 (0) (00:00:57)
Task 610 | 23:48:49 | Updating instance nginx: nginx/ca6c2391-685d-4e65-a3eb-2db32b436503 (0) (canary) (00:02:00)
                    L Error: 'nginx/ca6c2391-685d-4e65-a3eb-2db32b436503 (0)' is not running after update. Review logs for failed jobs: nginx
Task 610 | 23:50:49 | Error: 'nginx/ca6c2391-685d-4e65-a3eb-2db32b436503 (0)' is not running after update. Review logs for failed jobs: nginx
```

### 1.1. Bosh deploy
- Bosh Deploy 명령어는 Deployment Name, Release, 2개 이상의 Configs, Stemcell의 정보를 취합한 Main Deployment yml 파일 명세를 통해 실제 vSphere에서 VM를 생성하는 단계이며 실행 단계는 크게 아래와 같습니다.
	- Compile 단계: 명세한 Release를 통해 Package를 컴파일, 압축 해제 등이 이루어지는 단계, Release Compile 단계는 초기 1회에만 사용 됩니다.
	- Create 단계: 명세한 Stemcell OS를 이용하여 깡통 VM을 생성, 큰 변화가 없는 작업(manifest의 설정 변경에 대한 설정)은 1회만, Disk나 VM Type, Stemcell 버전 변경 등의 큰 작업은 Create -> Update 단계를 거치게 됩니다.
	- Update 단계: Release의 Job과 Package를 통해 실제 VM에 Component의 Monit,  Process 를 실행 하는 실행 단계


### 1.2. 해결 방법

#### 1.2.1.  Bosh Task Log 확인, 아래 라인에서 실패한 Job nginx을 확인 합니다.
-  Job의 실패는 VM이 생성 되었고 내부 Process를 실행하다 에러가 발생 됩니다.

```
Task 610 | 23:50:49 | Error: 'nginx/ca6c2391-685d-4e65-a3eb-2db32b436503 (0)' is not running after update. Review logs for failed jobs: nginx
```


#### 1.2.2. Error VM에 Bosh SSH 접속

```
$ bosh -d nginx ssh nginx/ca6c2391-685d-4e65-a3eb-2db32b436503
```

#### 1.2.3. Job의 Syslog에서 에러 확인
- Pivotal Cloud Foundry의 모든 Job은 아래 디렉토리에 Syslog가 생성이 됩니다.

```
# 기본
/var/vcap/sys/log/{JOB_NAME}

# 해당 Nginx TEST의 로그 확인
$ cd /var/vcap/sys/log/nginx
$ ls -al
-rw-r----- 1 root root 1095 Mar 27 04:35 pre-start.stderr.log
-rw-r----- 1 root root   26 Mar 27 04:35 pre-start.stdout.log

$ cat pre-start.stderr.log # 일부 발췌
+ echo '강제  ERROR CODE 삽입'
+ exit 1
```

#### 1.2.4. pre-start
- nginx release의 pre-start가 실행하며 에러를 발생한 것으로 확인 됩니다. pre-start의 동작 방법을 확인하여 에러 종료 `exit 1`을 제거
- Platform Bosh Deploy 중의 대부분 에러는 pre-start를 실행하며 네트워크 문제, VM Disk의 여유 공간, Bosh 내부의 통신 등으로 인하여 가장 많이 발생하는 에러입니다.
- 실습용 nginx deploy TEST의 pre-start는 manifest에 존재 합니다.


### 1.3. TIP
- VM Process 시작의 큰 동작은 아래와 같습니다. (거의 모든 Vm의 공통)
	- pre-start script: Process가 뜨기 전 File 다운로드 또는 Process Start 명령어가 존재하는 Script, 병렬 실행 (mysql을 예시로 DB Database와 Table을 생성하거나, service start nginx와 같은 추가 스크립트를 실행합니다.)
	- post-start-script: Process가 뜨고 난 후 해당 Process의 기능을 검증 하는 Script, 병렬 실행 (mysql process가 정상적인지 test select query를 실행한다.)
    - post-deploy-script: 모든 Job이 실행 후 전체 deploy에 대한 추가 스크립트를 실행 한다 , 병렬 실행

- 모든 실행 Script는 에러 대상 VM 내부의 /var/vcap/jobs/{JOB_NAME} 디렉토리에 존재 하고 있습니다.  (해당 Nginx Deploy Test 는 제외 됩니다)

- deploy VM 관련 주요 명령어

```
$ bosh -d nginx manifest # 전체 통합 된 full manifest file을 추출 할 수 있습니다.
$ bosh -d nginx deploy manifest.yml --recreate # 변경 및 설정 된 manifest 기반으로 모든 VM을 recreate 합니다.
$ bosh -d nginx deploy manifest.yml --fix # unresponsive agent 상태의 VM을 고칠 수 있습니다.
$ bosh -d nginx deploy manifest.yml --recreate-persistent-disks # VM의 persistent disk를 삭제 할 수 있습니다.
``` 

[수정 nginx_manifest 파일 바로 가기](http://git.posco.co.kr/projects/MES3/repos/platform-mgmt/browse/study/bosh/manifest.yml)

[첫 화면 바로가기](http://git.posco.co.kr/projects/MES3-PLATFORM/repos/study/browse/bosh/README.md)

