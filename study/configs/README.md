
# Bosh Configs

## 1. Bosh Configs Error 해결 방법

```
$ bosh -d nginx deploy manifest.yml
Task 608 | 23:32:20 | Error: Instance group 'nginx' references an unknown disk type 'default'
```

### 1.1. Bosh Configs
- configs는 현재 Bosh Deploy를 관리하는데에 있어 굉장히 중요한 역할을 담당합니다, Pivotal Opsmanager를 통해 자동으로 입력 되지만 Opensource 사용 시 수동 설정이 필요 할 만큼 중요한 속성입니다,
- configs의 종류는 크게 아래 3가지 입니다.
	- CPI Config: IaaS의 속성 중 요청에 대한 계정 정보, 물리 장비 정보를 결정
	- Cloud Config: IaaS의 속성 중 Network, Disk, LB 등의 Resource
	- Runtime Config: Bosh Deployment에 공통 적용 될 속성, Runtime Config에서 반드시 필요한 속성은 Bosh 관리 DNS 정보입니다. 
 
#### 1.1.1. CPI Config 예시
```
cpis:
- migrated_from:
  - name: ""
  name: dd6e31a9aff66e653df7
  properties:
    datacenters:
    - allow_mixed_datastores: true
      clusters:
      - mgmt: {}
      datastore_pattern: ^(xxxx|xxxx|xxx)$
      disk_path: xxxx
      name: xxxx/xxxx
      persistent_datastore_pattern: ^(xxx|xxxx|xxxx)$
      template_folder: xxxx
      vm_folder: xxxx
    default_disk_type: preallocated
    host: xxx.xxx.xxx.xxx
    password: xxxxxx
    user: xxxxx@xxxxx.local
  type: vsphere
```
 
#### 1.1.2. Cloud Config 예시
```
azs:
- cloud_properties:
    datacenters:
    - clusters:
      - mgmt:
          resource_pool: null
      name: mgmt
  cpi: dd6e31a9aff66e653df7
  name: mgmt
networks:
- name: mgmt
  subnets:
  - azs:
    - mgmt
    cloud_properties:
      name: xxxx.xxx
    dns:
    - xxx.xxx.xxx.xxx
    gateway: xxx.xxx.xxx.xxx
    range: xxx.xxx.xxx.xxx
    reserved:
    - xxx.xxx.xxx.xxx-xxx.xxx.xxx.xxx
    - xxx.xxx.xxx.xxx-xxx.xxx.xxx.xxx
    static:
    - xxx.xxx.xxx.xxx

```

#### 1.1.3. Runtime Config 예시
```
addons:
- include:
    stemcell:
    - os: ubuntu-trusty
    - os: ubuntu-xenial
  jobs:
  - name: bosh-dns
    release: bosh-dns
  name: bosh-dns
```

### 1.2. 해결 방법

#### 1.2.1. Manifest 속성 확인
- Bosh Deploy 대상의 Manifest 파일에는 Cloud Config 속성(network, azs, disk)등 에 따른 값을 설정을 해줘야합니다, Pivotal 제품에서는 Ops Manager가 관리하여 자동으로 이 값이 설정 되지만 Opensource는 일반 사용자들이 생성한 Manifest가 있어 해당 정보가 맞지 않습니다.
- nginx deploy manifest에서 사용 된 속성은 아래와 같습니다.

```
azs: [ test ] -> cloud-config의 azs 영역으로 변경 해야합니다.
vm_type: test -> cloud-config의 vm_typs 영역의 Name으로 변경 해야합니다.
persistent_disk_type: 1024 cloud-config의 disk_type 영역의 Name으로 변경 해야합니다.
networks:
- name: test cloud-config의 networks 영역의 Name으로 변경 해야합니다.
```
#### 1.2.2. Bosh 주요 Config 확인 방법

```
# 전체 검색
$ bosh configs
ID  Type     Name                     Team  Created At
6*  cloud    default                  -     2019-11-22 07:40:01 UTC
4*  cpi      default                  -     2019-11-22 07:31:11 UTC
2*  runtime  director_runtime         -     2019-11-22 07:31:11 UTC
1*  runtime  ops_manager_dns_runtime  -     2019-11-22 07:31:11 UTC

# 상세 검색
$ bosh config --name default --type cloud   
```

### 1.3. TIP
- 현재 Dyantrace 적용을 위해 runtime가 사용되고 있으며, Runtime Config 설정을 통해 모든 VM에 대한 일정 스크립트 실행 또는 Default 설정을 명시 할 수 있습니다.

[수정 nginx_manifest 파일 바로 가기](http://git.posco.co.kr/projects/MES3/repos/platform-mgmt/browse/study/bosh/manifest.yml)

[첫 화면 바로가기](http://git.posco.co.kr/projects/MES3-PLATFORM/repos/study/browse/bosh/README.md)

