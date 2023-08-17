## immutable
**프로그래밍에서 "immutable" 설정이나 객체는 한번 생성된 후에 그 상태가 변경되지 않도록 하는 것을 의미한다**

### **```immutable을 사용하는 이유```**
**예측 가능성:** 불변 객체는 상태가 변경되지 않기 때문에 예측하기 쉽고 버그가 발생할 확률이 줄어든다.

**스레드 안전:** 여러 스레드에서 동시에 접근해도 상태가 변경되지 않기 때문에 스레드 안전하다고 할 수 있다.

**함수형 프로그래밍:** 함수형 프로그래밍 패러다임에서는 불변성이 중요한 역할을 한다. 불변성을 유지하면서 함수들의 연산을 수행하게 된다.

## 컨테이너로 된 어플리케이션 배포 중 Docker hub의 rate limit 이슈로 인해 배포가 중단되었을 경우 해결할 수 있는 방법

**Private Registry 사용:** 자체 Docker registry를 설정하거나, 다른 클라우드 제공자의 container registry 서비스를 사용한다. 예를 들면, Amazon ECR, Google Container Registry, Azure Container Registry 등이 있다.

**Docker Hub 계정 업그레이드:** Docker Hub에서 유료 계정으로 업그레이드하여 rate limits를 높이거나 제거한다.

**인증 사용:** 로그인하지 않은 경우보다 로그인을 한 경우 rate limits가 더 높다. docker login 명령어를 사용하여 Docker Hub에 로그인한다.

**캐싱 활용:** CI/CD 파이프라인에 Docker layer 캐싱을 활용하여 이미 다운로드된 이미지 레이어의 재다운로드를 방지한다.

**요청 빈도 줄이기:** 배포 주기를 줄이거나, 필요할 때만 이미지를 pull하여 요청 빈도를 줄인다.

## 컨테이너 배포 중 exec /bin/sh: exec format error 해결 방법
**exec /bin/sh: exec format error 에러는 컨테이너를 실행할 때 발생하는데, 주로 호스트와 컨테이너 이미지의 아키텍처가 일치하지 않을 때 나타난다. 예를 들어, ARM 아키텍처용으로 빌드된 이미지를 x86 아키텍처의 시스템에서 실행하려고 할 때 이 에러가 발생할 수 있다.**

### **```해결 방법```**

**이미지 아키텍처 확인:** 사용하려는 이미지의 아키텍처를 확인한다. Docker Hub나 다른 컨테이너 레지스트리에서 이미지 정보를 통해 확인할 수 있다.

**적절한 이미지 사용:** 호스트 시스템의 아키텍처와 일치하는 이미지를 사용하도록 한다.

**멀티 아키텍처 이미지 빌드:** docker buildx 같은 도구를 사용하여 여러 아키텍처에 대해 이미지를 빌드한다. 이렇게 하면 실행 환경에 맞는 적절한 아키텍처의 이미지가 선택되어 실행된다.

**호스트 아키텍처 확인:** 호스트의 아키텍처를 확인하여 이미지 선택의 기준점으로 사용합니다. Linux에서는 uname -m 명령어로 확인할 수 있다.

## 1111 AWS account에 ECR repository 하나를 생성하였고, 2222 AWS account의 EKS에서 해당 이미지를 사용하고자 할 수 있는 방법

**ECR Repository 정책 수정: 1111 계정의 ECR 리포지터리에 정책을 추가하여 2222 계정의 EKS가 이미지를 pull 할 수 있는 권한을 부여한다.**
```
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "AllowPull",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<2222-account-id>:root"
      },
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:BatchCheckLayerAvailability"
      ]
    }
  ]
}
```
**2222 AWS 계정에서 권한 부여: 2222 계정에서 EKS가 사용하는 IAM role에 1111 계정의 ECR에 액세스할 수 있는 권한을 추가해야 한다.**
```
{
  "Effect": "Allow",
  "Action": [
    "ecr:GetDownloadUrlForLayer",
    "ecr:BatchGetImage",
    "ecr:BatchCheckLayerAvailability"
  ],
  "Resource": "arn:aws:ecr:<region>:<1111-account-id>:repository/<repository-name>"
}
```
**로그인 명령어 사용: 2222 계정의 EKS 노드에서 ECR 로그인 명령어를 실행하여 이미지를 pull 할 수 있도록 한다.**
```
$(aws ecr get-login --region <region> --no-include-email --registry-ids <1111-account-id>)

```