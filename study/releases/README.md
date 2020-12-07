# Bosh Releases

## 1. Bosh Releases Error 해결 방법

```
$ bosh -d nginx deploy manifest.yml
Task 603 | 23:28:38 | Error: Release 'nginx' doesn't exist
```

### 1.1. Bosh Release
- Bosh가 관리하는 Deployment 대상에 어떠한 Vm을 동작하려면 반드시 Release가 필요합니다.
- Release는 Bosh Director 위에 설치 되는 Software의 동작이나, 연관 관계에 대해 정의합니다.
- Release는 아래 기본 구성 요소를 포함 합니다.(사용 되는 VM 별 상이 할 수 있습니다.)
	- Jobs: Release에 Packaging 되어 있는 Service나 Application에 대한 Spec, 실행 방법(실행 Script)을 정의
	- Packages: Job에 대한 Source Code 또는 Dependency, 실행 바이너리 파일 정의



### 1.2. 해결 방법
#### 1.2.1. Manifest 설정 확인
- manifest 파일의 Releases Block은 아래와 같습니다.

```
releases:
- name: nginx
  version: latest <- 만약 latest가 아닌 버전으로 명시 되어 있는 경우 반드시 해당 버전으로 Release를 업로드해야합니다.
```

- 해당 nginx Release를 bosh cli를 실행 시킬 수 있는 환경으로 Download 합니다.
	- [nginx release download 하기](https://bosh.io/d/github.com/cloudfoundry-community/nginx-release?v=1.17.0) 


#### 1.2.2. Bosh Upload-Release 요청
- 해당 명령어를 통해 Upload 된 Release는 Bosh Director의 Blobstore에 저장됩니다.
```
$ bosh upload-release nginx-release-1.17.0.tgz
```

### 1.3. TIP
- Pivotal/OpenSource Bosh에 Release 되어 있는 Product를 제외하고 Ubuntu/Windows/CentOS 기반의 모든 Product는 Release로 Package 할 수 있으며, Bosh를 통해 관리가 가능하게 할 수 있습니다.
- Release 생성 관련 뼈대를 만드는 주요 명령어는 아래와 같습니다.

```
$ bosh init-release # Release의 구조 디렉토리를 생성합니다.
$ bosh create-release # Relesae의 구조를 통해 Tarball을 패키징합니다.
```

- 인터넷이 되는 외부 환경에서는 wget 형태의 URL를 입력 가능 합니다.

```
$ bosh upload-release https://bosh.io/d/github.com/DataDog/datadog-agent-boshrelease?v=2.13.7180
```
- [Download가 가능한 Opensource Release 집합](https://bosh.io/releases/)

- 만약 수동으로 Release Tarball를 생성한다면, 반드시 Checksum을 맞춰야합니다.


[수정 nginx_manifest 파일 바로 가기](http://git.posco.co.kr/projects/MES3/repos/platform-mgmt/browse/study/bosh/manifest.yml)

[첫 화면 바로가기](http://git.posco.co.kr/projects/MES3-PLATFORM/repos/study/browse/bosh/README.md)

