# Bosh Deploy

- Opensource, Pivotal Bosh VM Deploy를 사용하며 발생 할 수 있는 주요 에러와 VM 배포에 대한 중요 요소, 에러 해결방법, 기타 유용한 정보를 설명합니다.


## STEP 1. Bosh Deployment Name Error

```
$ bosh -d nginx-test deploy manifest.yml
Expected manifest to specify deployment name 'nginx2' but was 'nginx'
```

해결 방법 및 설명: [바로가기]()


## STEP 2. Bosh Releases Error

```
$ bosh -d nginx deploy manifest.yml
Task 603 | 23:28:38 | Error: Release 'nginx' doesn't exist
```

해결 방법 및 설명: [바로가기]()

## STEP 3. Bosh Configs Error

```
$ bosh -d nginx deploy manifest.yml
Task 608 | 23:32:20 | Error: Instance group 'nginx' references an unknown disk type 'default'
```

해결 방법 및 설명: [바로가기]()


## STEP 4. Bosh Stemcells Error

```
$ bosh -d nginx deploy manifest.yml
Task 609 | 23:42:35 | Preparing deployment: Preparing deployment (00:00:00)
                    L Error: Stemcell version '456.16' for OS 'ubuntu-xenial' doesn't exist
Task 609 | 23:42:35 | Error: Stemcell version '456.16' for OS 'ubuntu-xenial' doesn't exist
```

해결 방법 및 설명: [바로가기]()

## STEP 5. Bosh Deploy Error - 트러블 슈팅

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

해결 방법 및 설명: [바로가기]()


## STEP 6. Bosh Deploy 정상 배포

```
$ bosh -d nginx deploy manifest.yml
Task 643 | 00:32:31 | Preparing deployment: Preparing deployment (00:00:02)
Task 643 | 00:32:33 | Preparing deployment: Rendering templates (00:00:00)
Task 643 | 00:32:33 | Preparing package compilation: Finding packages to compile (00:00:00)
Task 643 | 00:32:33 | Updating instance nginx: nginx/855c60a5-fbd8-4dc2-890f-91c5e1e0f3d5 (0) (canary) (00:00:23)
```

- bosh vms 형상을 확인하여 해당 VM IP를 웹 브라우저로 확인

```
$ bosh vms
Task 659. Done

Deployment 'nginx'

Instance                                    Process State  AZ    IPs             VM CID                                   VM Type  Active  Stemcell
nginx/855c60a5-fbd8-4dc2-890f-91c5e1e0f3d5  running        mgmt  ${IP}  vm-c7b7ea29-9fa5-4105-b1fb-f74fb41bc1a3  nano     true    -

1 vms
```

- 정상 deploy 성공 후 결과

![ex_screenshot](./images/hello.PNG)

## STEP 7. Bosh Deploy VM 내부 설정 편집

설명: [바로가기]()

- nginx config 설정 편집 후 결과

![ex_screenshot](./images/hello.PNG)

