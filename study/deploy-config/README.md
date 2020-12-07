

# Bosh Deploy VM 내부 설정 편집

## 1. Bosh Deploy VM 내부 설정 편집

- VM의 내부 설정은 "Ops Manager UI에서 Apply Change" 시 원복 됩니다.
- Component의 특별한 설정(화면에서 변경 하지 못하는 설정)을 건들이며 확인하는 방법이 될 수 있습니다.
- 간단한 트러블 슈팅의 기능으로 사용 될 수 있습니다.

```
예시 1) Metrics App과 PAS 버전 호환 문제로 내부에서 Node Buildpack Versin을 강제적으로 명시하여 사용 하였습니다.
예시 2) Choas Test 기능을 검증하기 위해 UI에서 변경 불가능한 Go Router의 Route Table 정리 기간 설정을 편집 & 적용 하였습니다.
예시 3) Dynatrace의 agent Download URL 404 예외에 대한 에러를 확인 하였습니다.
```
### 1.1. 설정 변경 방법

#### 1.1.1.  변경 대상의 Job Config File
- Platform의 거의 대부분의 Release들은 Config 설정 파일, mysql -> my.cnf, nginx -> nginx.conf 같은 설정 파일은 /var/vcap/job/{JOB_NAME} 디렉토리에 존재 하고 있습니다.
- 해당 디렉토리의 config 파일에서 Sofeware의 Default 설정을 확인 할 수 있습니다.

#### 1.2.2. 대상 VM에 Bosh SSH 접속

```
$ bosh -d nginx ssh nginx/ca6c2391-685d-4e65-a3eb-2db32b436503
```

#### 1.2.3. Job의 Config 확인
- Bosh Nginx Deploy TEST Job의 nginx.conf의 위치는 아래와 같습니다.

```
/var/vcap/jobs/nginx/etc/nginx.conf

# 해당 설정 파일 중 root 디렉토리를 변경 합니다.
# 변경 전
root /var/vcap/data/nginx/document_root;
# 변경 후
root /var/vcap/data/nginx/document_root-2;
```

#### 1.2.4. monit restart
- 대상의 job monit process를 재실행 합니다.

```
$ monit summary
The Monit daemon 5.2.5 uptime: 3h 21m

Process 'nginx'                     running
Process 'bosh-dns'                  running
Process 'bosh-dns-resolvconf'       running
Process 'bosh-dns-healthcheck'      running
System 'system_localhost'           running

$ monit restart nginx
```

### 1.3. TIP
- Bosh Agent를 통해 관리되는 Process 종류는 아래와 같습니다.
  - monit: VM에 설치 되는 Bosh Agent를 통해 관리 되는 Release Job의 Process 목록입니다. Monit를 통해서 각 Process를 시작/종료/재시작/설정 반영 할 수 있습니다.

	```
	$ monit summary
	The Monit daemon 5.2.5 uptime: 2d 19h 9m

	Process 'nginx'                     running
	Process 'bosh-dns'                  running
	Process 'bosh-dns-resolvconf'       running
	Process 'bosh-dns-healthcheck'      running
	System 'system_localhost'           running
	```
  - bpm: bpm은 bosh agent와 monit 사이의 계층으로써 Monit의 Process 순서 또는 시작 시간을 정의하며 실제 pid status를 관리 합니다.

	   ```
	$ bpm list
	Name                        Pid   Status
	bosh-dns-adapter            18001 running
	iptables-logger             18079 running
	loggr-forwarder-agent       17779 running
	loggr-syslog-agent          17706 running
	loggr-udp-forwarder         17852 running
	loggregator_agent           17164 running
	```

[수정 nginx_manifest 파일 바로 가기](http://git.posco.co.kr/projects/MES3/repos/platform-mgmt/browse/study/bosh/manifest.yml)

[첫 화면 바로가기](http://git.posco.co.kr/projects/MES3-PLATFORM/repos/study/browse/bosh/README.md)

