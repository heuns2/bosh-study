#  Bosh Release Concept

- Bosh Director를 통해 VM(Software)을 배포에 필요한 Release의 개념 & 방법에 대해 기술한 문서이다.
- Bosh CLI v2를 기준으로 작성하였다.

## 1. Bosh Release Core Concept

### 1.1. Creating a Release
- Release는 Bosh Director 위에 설치 되는 Software의 동작이나, 연관 관계에 대해 정의 한다.
- Release는 아래 기본 구성 요소를 포함 한다.
	- Jobs: Release에 Packaging 되어 있는 Service나 Application에 대한 Spec, 실행 방법을 정의 
	- Packages: Job에 대한 Source Code 또는 Dependency 정의
	- Source: Packages에서 사용하는 바이너리 파일이 아닌 파일 정의
	- Blob: Packages에서 사용하는 바이너리 파일을 정의

#### 1.1.1. Create the release directory
- Bosh CLI를 통해 Release Directory를 생성 한다.
- git을 초기화 하기 위해서 --git sub command를 실행.한다
```
$ bosh init-release --dir {release_name} --git
Succeeded
```
- Release Director의 Tree 구조를 확인 한다.
```
$ cd {release_name}
$ tree . 
.
├── config
│   ├── blobs.yml
│   └── final.yml
├── jobs
├── packages
└── src

4 directories, 2 files
```
- 해당 Directory는 Bosh Director가 Release를 통해 VM Create Compile 단계 중 /var/vcap/jobs, packages, src, blobs로 나타난다.

#### 1.1.2. Create Job Skeletons
- Bosh CLI를 통해 특정 Software의 Job에 대한 Skeletons 구성을 생성 한다.
```
$ bosh generate-job {job_name}
Succeeded
```
- Release Director의 Tree 구조를 확인 한다. 
```
$ tree .
.
├── config
│   ├── blobs.yml
│   └── final.yml
├── jobs
│   └── test-exec # 생성
│       ├── monit # 생성
│       ├── spec # 생성
│       └── templates # 생성
├── packages
└── src

6 directories, 4 files
```
#### 1.1.3.  Create control scripts
- 특정 Job을 Start/Stop하는 Control Script를 작성하고 Job Director의 Monit에 반영 한다.
- Monit은 Bosh Agent가 관리하는 제어 스크립트이며 상태를 모니터링하는 요소이다.
- 특정 Software나 VM에 대한 Dependency 또는 Jar, War 파일을 실행하는 Script로 반영 & 변경 할 수 있다.

```
#!/bin/bash

RUN_DIR=/var/vcap/sys/run/web_ui
LOG_DIR=/var/vcap/sys/log/web_ui
PIDFILE=${RUN_DIR}/pid

case $1 in

  start)
    mkdir -p $RUN_DIR $LOG_DIR
    chown -R vcap:vcap $RUN_DIR $LOG_DIR

    echo $$ > $PIDFILEa

    cd /var/vcap/packages/ardo_app

    export PATH=/var/vcap/packages/ruby_1.9.3/bin:$PATH

    exec /var/vcap/packages/ruby_1.9.3/bin/bundle exec \
      rackup -p <%= p('port') %> \
      >>  $LOG_DIR/web_ui.stdout.log \
      2>> $LOG_DIR/web_ui.stderr.log

    ;;

  stop)
    kill -9 `cat $PIDFILE`
    rm -f $PIDFILE

    ;;

  *)
    echo "Usage: ctl {start|stop}" ;;

esac
```

#### 1.1.4.  Update Monit files

- Bosh Release를 동작 및 상태를 확이하는 중요한 요소인 Monit Script를 작성한다. 
- Monit Script가 없을 경우 Bosh Director가 에러를 감지 하지 못 한다.
- Monit File은 아래를 포함 한다.
	- Job의 Process ID(PID)를 지정
	- Job의 시작과 종료 Script 명령을 참조

```
check process web_ui
  with pidfile /var/vcap/sys/run/web_ui/pid
  start program "/var/vcap/jobs/web_ui/bin/ctl start"
  stop program "/var/vcap/jobs/web_ui/bin/ctl stop"
  group vcap
```

#### 1.1.5. Update job specs
- Bosh Deploy 단계 중 Complie에서의 실행 및 Job의 Config를 정의하는 Template 파일을 작성 한다.
```
templates:
  ctl.erb: bin/ctl
```
-   Key는 Template의 이름을 나타낸다.
-   Value는 VM에서의 실행 위치를 나타 낸다.



