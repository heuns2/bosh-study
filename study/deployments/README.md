
# Bosh Deployments

## 1. Bosh Deployment Name Error 해결 방법

```
$ bosh -d nginx-test deploy manifest.yml
Expected manifest to specify deployment name 'nginx2' but was 'nginx'
```

### 1.1. Bosh Manifest Block

- Bosh를 통한 모든 배포는 Yaml 형식의 Manifest를 사용하여 배포하게 됩니다. 모든 Bosh Deployment Manifest 파일은 아래와 같은 규칙을 가지고 있습니다.

```
Deployment Identification: 배포 명을 설정하는 Block
Releases Block: 배포 시 사용할 Release 명과 버전을 설정하는 Block
Networks Block: 배포 시 할당 할 IaaS의 네트워크 정보를 설정하는 Block
Resource Pools Block: BOSH가 설치 하고 관리 하는 VM의 속성 Block
Disk Pools Block: BOSH가 작성하고 관리하는 Disk Pool의 등록 정보를 설정하는 Block
Compilation Block: VM의 컴파일 속성을 설정하는 Block
Update Block: 배포 중에 BOSH가 작업 인스턴스를 업데이트하는 방법을 정의하는 Block
Jobs Block: Job에 대한 구성 및 자원을 설정하는 Block
Properties Block: 전역 속성과 일반화 된 구성 정보(config)를 설정 하는 Block
```

### 1.2. 해결 방법

- Bosh Nginx를 배포하며 발생 한 Deployment은 아래와 같습니다.

```
nginx_manifest deploy manifest 파일의 2 line 
name: nginx <------ deployment block의 deployment name
```

- 만약 해당 에러를 해결하고자 한다면 `name: nginx`를 `$ bosh -d nginx-test deploy manifest-error.yml` 명령어의 nginx-test와 맞춰주거나 명령어를 nginx-test -> nginx로 변경해야합니다.

#### 해결 방법 1) manifest의 deployment name 변경
```
name: nginx -> name: nginx-test # 변경
$ bosh -d nginx-test deploy manifest.yml
```

#### 해결 방법 2) manifest 변경 없이 bosh deploy cli 변경
```
$ bosh -d nginx deploy manifest.yml
```

### 1.3. TIP

#### bosh deployment name은 bosh가 관리하는 VM 목록의 최상위 검색 Index이며 명칭의 중복 불가, 해당 deployment의 중복 요청이 불가능 합니다, 추 후 API, Bosh 명령어의 최상의 검색 및 실행 리소스 기반이 됩니다.

- bosh cli를 통하여 사용되는 주요 기능은 아래와 같습니다.

```
$ bosh -d {deployment_name} vms # deployment로 묶여진 VM의 형상 관리
$ bosh -d {deployment_name} is # deployment로 묶여진 Instnace Group의 형상 관리
$ bosh -d {deployment_name} deld # deployment의 삭제
$ bosh -d {deployment_name} deploy # deployment의 배포
```

[수정 nginx_manifest 파일 바로 가기](http://git.posco.co.kr/projects/MES3/repos/platform-mgmt/browse/study/bosh/manifest.yml)

[첫 화면 바로가기](http://git.posco.co.kr/projects/MES3-PLATFORM/repos/study/browse/bosh/README.md)

