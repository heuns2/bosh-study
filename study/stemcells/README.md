
# Bosh Stemcells

## 1. Bosh Stemcells Error 해결 방법

```
$ bosh -d nginx deploy manifest.yml
Task 609 | 23:42:35 | Preparing deployment: Preparing deployment (00:00:00) L Error: Stemcell version '456.16' for OS 'ubuntu-xenial' doesn't exist
Task 609 | 23:42:35 | Error: Stemcell version '456.16' for OS 'ubuntu-xenial' doesn't exist
```

### 1.1. Bosh Stemcells
- Stemcell은 IaaS의 특정 패키징으로 포장 된 버전 관리 OS 이미지로 기본 OS 이미지의 구성을 가지고 있습니다. 또한 Bosh를 통해 배포하는 VM의 Base OS로 사용됩니다.
- 모든 IaaS 스템셀에는 Bosh Agent를 포함하고 있으며 Agent를 통해 해당 Bosh의 Blobstore로부터 Job을 배포합니다.
- Stemcell의 주요 패키징 정보는 아래와 같습니다.

- IaaS 관련 프로퍼티
```
stemcell_formats:
- vsphere-ova
- vsphere-ovf
cloud_properties:
  name: bosh-vsphere-esxi-ubuntu-xenial-go_agent
  version: '315.141'
  infrastructure: vsphere
  hypervisor: esxi
  disk: 3072
  disk_format: ovf
  container_format: bare
  os_type: linux
  os_distro: ubuntu
  architecture: x86_64
  root_device_name: "/dev/sda1"
```

- OS 관련 패키지 목록 (약 400개)
```
ii  adduser                               3.113+nmu3ubuntu4                          all          add and remove users and groups
  7 ii  amd64-microcode                       3.20180524.1~ubuntu0.16.04.2               amd64        Processor microcode firmware for AMD CPUs
  8 ii  anacron                               2.3-23                                     amd64        cron-like program that doesn't go by time
  9 ii  apparmor                              2.10.95-0ubuntu2.11                        amd64        user-space parser utility for AppArmor
 10 ii  apparmor-utils                        2.10.95-0ubuntu2.11                        amd64        utilities for controlling AppArmor
 11 ii  apt                                   1.2.32                                     amd64        commandline package manager
 12 ii  apt-utils                             1.2.32                                     amd64        package management related utility programs
 13 ii  auditd                                1:2.4.5-1ubuntu2.1                         amd64        User space tools for security auditing
 14 ii  autotools-dev                         20150820.1                                 all          Update infrastructure for config.{guess,sub} files
 15 ii  base-files                            9.4ubuntu4.11                              amd64        Debian base system miscellaneous files
 16 ii  base-passwd                           3.5.39                                     amd64        Debian base system master password and group files
 17 ii  bash                                  4.3-14ubuntu1.4                            amd64        GNU Bourne Again SHell
 18 ii  bind9-host                            1:9.10.3.dfsg.P4-8ubuntu1.15               amd64        Version of 'host' bundled with BIND 9.X
 19 ii  binutils                              2.26.1-1ubuntu1~16.04.8                    amd64        GNU assembler, linker and binary utilities
 20 ii  bison                                 2:3.0.4.dfsg-1                             amd64        YACC-compatible parser generator
 21 ii  bsdmainutils                          9.0.6ubuntu3                               amd64        collection of more utilities from FreeBSD
 22 ii  bsdutils                              1:2.27.1-6ubuntu3.9                        amd64        basic utilities from 4.4BSD-Lite
 23 ii  build-essential                       12.1ubuntu2                                amd64        Informational list of build-essential packages
 24 ii  bzip2                                 1.0.6-8ubuntu0.2                           amd64        high-quality block-sorting file compressor - utilities
 25 ii  ca-certificates                       20170717~16.04.2                           all          Common CA certificates
 26 ii  chrony                                2.1.1-1ubuntu0.1                           amd64        Versatile implementation of the Network Time Protocol
 27 ii  cloud-guest-utils                     0.27-0ubuntu25.1                           all          cloud guest utilities
 28 ii  cmake                                 3.5.1-1ubuntu3                             amd64        cross-platform, open-source make system
 29 ii  cmake-data                            3.5.1-1ubuntu3                             all          CMake data files (modules, templates and documentation)
 30 ii  console-setup                         1.108ubuntu15.5                            all          console font and keymap setup program
 31 ii  console-setup-linux                   1.108ubuntu15.5                            all          Linux specific part of console-setup
 32 rc  console-setup-mini                    1.108ubuntu15.5                            all          console font and keymap setup program - reduced version for Linu    x
..... 생략 생략 생략.....
```

- Version이 Upgrade 될 때, CVE 보안 패치와 Bosh Agent와의 상호 작용 Upgrade가 있습니다.


### 1.2. 해결 방법

#### 1.2.1. Manifest 설정 확인
-   manifest 파일의 Stemcell Block은 아래와 같습니다.

```
stemcells:
- alias: ubuntu
  os: ubuntu-xenial
  version: "456.16" <- 만약 latest가 아닌 버전으로 명시 되어 있는 경우 반드시 해당 버전으로 Stemcell을 업로드해야 하며 오래 전 개발 된 소스코드에 대해서는 최신 버전의 OS 커널 버전이 지원이 안될 수 있습니다.
```

-   해당 버전의 Stemcell을 bosh cli를 실행 시킬 수 있는 환경으로 Download 합니다.
    -   [Stemcell v456.16 download 하기](https://s3.amazonaws.com/bosh-core-stemcells/456.16/bosh-stemcell-456.16-vsphere-esxi-ubuntu-xenial-go_agent.tgz)

#### 1.2.2. Bosh Upload-Stemcell 요청

-   해당 명령어를 통해 Upload 된 Stemcell은 Bosh Director의 Blobstore에 저장됩니다.

```
$ bosh upload-stmecell bosh-stemcell-456.16-vsphere-esxi-ubuntu-xenial-go_agent.tgz
```


### 1.3. TIP
- Ops Manager Upgrade & 복구 시 Stemcell 유실 발생 또는 vSphere Disk Type 변경 시 bosh cli 아래 옵션을 통해 교체가 가능합니다.

```
$ bosh upload-stmecell bosh-stemcell-456.16-vsphere-esxi-ubuntu-xenial-go_agent.tgz --fix
```

- 인터넷이 되는 외부 환경에서는 wget 형태의 URL를 입력 가능 합니다.

```
$ bosh upload-stemcell https://bosh-core-stemcells.s3-accelerate.amazonaws.com/621.61/bosh-stemcell-621.61-vsphere-esxi-ubuntu-xenial-go_agent.tgz
```
- [Download가 가능한 Opensource Stemcell 집합](https://bosh.io/stemcells/)
- [Download가 가능한 Pivotal Stemcell 집합](https://network.pivotal.io/products/stemcells-ubuntu-xenial/)

- 만약 수동으로 Stmecell Tarball를 생성한다면, 반드시 Checksum을 맞춰야합니다.


[수정 nginx_manifest 파일 바로 가기](http://git.posco.co.kr/projects/MES3/repos/platform-mgmt/browse/study/bosh/manifest.yml)

[첫 화면 바로가기](http://git.posco.co.kr/projects/MES3-PLATFORM/repos/study/browse/bosh/README.md)

