---
title: AWS Systems Manager의 Session Manager를 이용하여 프라이빗 환경의 서버 접근통제 구현
authors:
  - dohyeon-lee
date: 2022-09-29T09:19:35.960Z
categories:
  - Hands On
tags:
  - SessionManager
---
## 목차 <br>

> :one: [서버 접근통제란](#서버-접근통제란)
>
> :two: [Session Manager란](#session-manager란)
>
> > 2-1 [계획](#계획)
>
> > 2-2 [아키텍처](#아키텍처)
>
> :three: [Session Manager 설정하기](#session-manager-설정하기)
>
> > 3-1 [사전 조건](#사전-조건)
>
> > 3-2 [엔드포인트 설정](#엔드포인트-설정)
>
> > 3-3 [EC2 설정](#ec2-설정)
>
> > 3-4 [IAM 설정](#iam-설정)
>
> > 3-5 [S3 logging 설정](#s3-logging-설정)
>
> > 3-6 [로그 확인](#로그-확인)
>
> :four: [마치며](#마치며)

- - -

안녕하세요. 클라우드메이트 TE팀 이도현입니다. <br> AWS의 서비스 중 간편하게 서버 접근통제를 구현할 수 있는 *Session Manager*에 대해 공유하고자 합니다. <br> 테스트하실 수 있게 핸즈온으로 구성했습니다. <br><br>

## 서버 접근통제란

접근통제란 적절한 권한이 인가된 사용자만 해당 시스템에 접근할 수 있도록 하고, 권한이 없는 사용자는 접근하지 못하도록 통제하는 것입니다. <br>

또한 인가된 사용자의 부인 방지를 위해 작업 이력에 대한 로깅을 할 수 있어야 합니다. <br><br>

## Session Manager란

간단하게 Session Manager의 기능에 대해 설명드리겠습니다. <br>

Session Manager는 Systems Manager의 기능 중 하나로써, 
EC2 인스턴스, 엣지 디바이스, 온프레미스 서버 및 가상 머신을 브라우저 기반 셸 또는 AWS CLI를 통해 관리할 수 있습니다. <br>

또한 Session Manager는 4가지 장점이 있습니다. <br>

* 인바운트 포트를 열 필요가 없습니다. <br>
* Bastion 호스트를 사용하지 않습니다. <br>
* SSH 키를 사용하지 없습니다. <br>
* 세션 활동을 로깅 및 감사가 가능합니다. 

<br>

로깅 및 감사 기능은 다음과 같은 AWS 서비스와의 통합을 통해 제공됩니다. <br>

* S3
* CloudTrail
* CloudWatch Logs
* EventBridge 및 SNS 

<br>

### 계획

> :one: 인터넷이 되지 않는 Private 환경을 구성합니다. <br> :two: 특정 사용자만 특정 EC2 인스턴스에 접근이 가능하도록 합니다. <br> :three: MFA 인증이 되지 않으면 접근하지 못하도록 합니다. <br> :four: 사용자가 인스턴스에 접근할 때 접근 이력과 명령어 실행 이력이 로깅되도록 합니다. <br>

IAM 정책을 이용하여 특정 사용자에게만 권한을 부여하고 특정 인스턴스에만 접근할 수 있도록 합니다. <br>

또한 Session Manager의 장점을 좀 더 살리고 싶어 보안성을 높여 폐쇄망에서 사용하는 것을 전제로 하고 MFA 인증을 강제하였습니다. <br>

그리고 인스턴스 내부의 명령어 실행 이력을 S3 버킷에 로깅하도록 하겠습니다. <br><br>

### 아키텍처

![아키텍처](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\architecture.png)
<br>
편의상 User를 Admin으로 표시하였지만, <br>
테스트 환경에서는 User A는 EC2-A에만 User B는 EC2-B에만 접근할 수 있도록 구성하겠습니다. <br><br>
Session Manager의 작동 방식을 잠시 살펴보겠습니다. <br>
사용자가 세션을 시작하라는 첫 번째 명령을 보내면, <br>
Session Manager 서비스에서 사용자의 ID를 인증하고, <br>
IAM 정책에 따라 사용자에게 부여된 권한을 확인한 다음 구성 설정을 확인하고, <br>
SSM Agent에 양방향 연결을 열라는 메시지를 보냅니다. <br>
연결이 설정되어 사용자가 다음 명령을 입력하면 SSM Agent의 명령 출력이 이 통신 채널로 업로드되고 다시 사용자의 로컬 시스템으로 전송됩니다. <br><br>

## Session Manager 설정하기

이 글에서는 VPC를 생성하는 설명은 생략하도록 하겠습니다. <br>
실습을 진행하는 환경은 인터넷 게이트웨이 또는 NAT 게이트웨이가 없는 프라이빗 서브넷에서 이루어집니다. <br><br>

### 사전 조건

Session Manager 사용을 시작하기 전에 환경이 다음 요구 사항을 충족하는지 사전 조건을 확인해 보아야 합니다. <br>

> :one: 지원되는 운영체제인지 확인
>
> * https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/prereqs-operating-systems.html 
>
> :two: 관리형 노드에 SSM Agent가 설치되어 있는지 확인
>
> * https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/session-manager-prerequisites.html 
>
> :three: 엔드포인트에 연결
>
> * ec2messages.*region*.amazonaws.com
> * ssm.*region*.amazonaws.com
> * ssmmessages.*region*.amazonaws.com
> * s3.*region*.amazonaws.com 
>
>   * *S3 로깅을 위한 게이트웨이 엔드포인트 추가*

ec2messages, ssm, ssmmessages의 엔드포인트를 생성하는 것은 Private 환경에서 사용하기 위한 *필수 조건*입니다. <br>
여기서는 S3 버킷에 로깅하기 위해 S3 게이트웨이 엔드포인트까지 추가적으로 생성하겠습니다. <br><br>

### 엔드포인트 설정

3가지 서비스에 대해 엔드포인트 생성을 한 후에, <br>
해당 엔드포인트들의 보안 그룹에 HTTPS 인바운드 트래픽을 허용해 줘야 합니다. <br><br>
먼저 엔드포인트용 보안 그룹을 생성하겠습니다. <br>
![엔드포인트\_보안그룹](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\endpoint1.png) <br>
인바운드 규칙에 자신의 VPC 대역을 소스로 두고 443 포트를 허용해 줍니다. <br><br>

엔드포인트 3개를 각각 하나씩 생성합니다. <br>
![엔드포인트\_생성\_1](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\endpoint2.png) <br>
![엔드포인트\_생성\_2](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\endpoint3.png) <br>
위에서 만든 엔드포인트용 보안 그룹을 적용해 줍니다. <br>

위와 같은 방법으로 *ec2messages, ssm, ssmmessages* 엔드포인트를 생성합니다. <br>

S3 게이트웨이 엔드포인트도 생성합니다. <br>
![엔드포인트\_생성\_3](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\endpoint4.png) <br>
EC2 인스턴스가 존재하는 서브넷이 연결되어 있는 라우팅 테이블을 선택합니다. <br>

필요한 엔드포인트를 모두 생성하였습니다. <br>
![엔드포인트\_생성\_4](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\endpoint5.png) <br><br>

### EC2 설정

먼저 EC2용 보안 그룹을 생성하겠습니다. <br>
![EC2_1](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_1.png) <br>
아웃바운드 규칙은 interface 엔드포인트들이 포함되어 있는 서브넷 대역에 443 포트를 허용합니다. <br>
로깅을 위해 s3 게이트웨이 엔드포인트를 대상으로도 443 포트를 허용합니다.  <br>

다음으로 프라이빗 서브넷에 EC2 인스턴스를 생성합니다. <br>
![EC2_2](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_2.png) <br>
인스턴스를 생성할 때 *네트워크 설정 > 기존 보안 그룹 선택 > 위에서 만든 EC2용 보안 그룹을 선택*합니다. <br>

특정 사용자만 특정 인스턴스에 접근하는 것을 테스트하기 위해 2대의 인스턴스를 생성했습니다. <br>
![EC2_3](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_3.png) <br>

그리고 세션 매니저를 사용하기 위해 EC2에 IAM Role을 설정해 주어야 합니다. <br>
*IAM > 역할 > 역할 생성*을 통해 설정합니다. <br>
![EC2_4](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_4.png) <br>
![EC2_5](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_5.png) <br>
AWS 관리형 정책인 Amazon SSM Managed Instance Core를 꼭 포함시켜야 합니다. <br>
이 관리형 정책은 Systems Manager 서비스 핵심 기능을 사용하도록 설정하는 EC2 역할 정책입니다. <br>
그리고 해당 역할에 인라인 정책을 추가하여 EC2 인스턴스가 로깅할 특정 S3 버킷에 쓰기 권한을 갖도록 설정합니다. <br>
![EC2_6](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_6.png) <br>

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LoggingS3",
            "Effect": "Allow",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::[YOUR_S3_BUCKET_NAME]/*"
        }
    ]
}
```

다시 EC2 콘솔로 돌아와 EC2 인스턴스 2대 모두 IAM 역할을 수정해 줍니다. <br>
해당 EC2를 선택 후 *작업 > 보안 > IAM 역할 수정*을 클릭하여 위에서 만들어두었던 IAM 역할을 선택한 후 IAM 역할 업데이트를 클릭합니다. <br>
![EC2_7](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ec2_7.png) <br>
IAM 역할 업데이트를 진행했다면 꼭 해당 인스턴스를 *재부팅*해 줍니다. <br><br>

### IAM 설정

IAM 사용자 2명을 추가해 줍니다. <br>

* dohyeon_A <br>
* dohyeon_B <br>

그리고 각각 새로 정책을 생성 후 사용자에게 권한을 추가해 줍니다. <br>

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:Describe*",
                "ssm:Get*",
                "ssm:List*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:StartSession"
            ],
            "Resource": [
                "arn:aws:ec2:[region]:[YOUR_ACCOUNT_NUMBER]:instance/*"
            ],
            "Condition": {
                "StringEquals": {
                    "ssm:resourceTag/[KEY]": [
                        "[VALUE]"
                    ]
                },
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:TerminateSession",
                "ssm:ResumeSession"
            ],
            "Resource": [
                "arn:aws:ssm:*:*:session/${aws:username}-*"
            ]
        }
    ]
}
```

Condition의 `ssm:resourceTag/[KEY]`와 `[VALUE]`는 사용자가 접근하고자 하는 EC2 인스턴스의 태그 Key와 Value를 뜻합니다. <br>
`aws:MultiFactorAuthPresent`를 `true`로 설정하여 MFA 인증을 해야 접근할 수 있도록 합니다. <br>

그리고 추가적으로 두 사용자 모두 동일한 정책을 추가하여 MFA 디바이스를 삭제하거나 추가하고 패스워드를 바꿀 수 있도록 설정합니다. <br>
또한 로그인 후 MFA 인증을 하지 않으면 나열해놓은 서비스를 제외하고는 접근할 수 없도록 설정합니다. <br>

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowViewAccountInfo",
            "Effect": "Allow",
            "Action": [
                "iam:ListVirtualMFADevices",
                "iam:ListUsers",
                "iam:GetAccountPasswordPolicy",
                "iam:GetAccountSummary"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowChangeOwnPasswordsOnFirstLogin",
            "Effect": "Allow",
            "Action": [
                "iam:ChangePassword",
                "iam:GetUser"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowChangeOwnPasswordsAfterMFAEnabled",
            "Effect": "Allow",
            "Action": [
                "iam:GetLoginProfile",
                "iam:UpdateLoginProfile"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnVirtualMFADevice",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice"
            ],
            "Resource": "arn:aws:iam::*:mfa/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnUserMFA",
            "Effect": "Allow",
            "Action": [
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "DenyAllExceptListedIfNoMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice",
                "sts:GetSessionToken",
                "iam:ListUsers",
                "iam:ChangePassword"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

<br>

### S3 logging 설정

*Systems Manager > 세션 관리자 > 기본 설정 > 편집*을 클릭하여 S3 logging 설정을 합니다. <br>
![ssm1](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\ssm1.png) <br><br>

### 로그 확인

이후 생성한 사용자에 로그인 후 MFA 디바이스 할당을 합니다. <br>
로그아웃 후 MFA 인증을 하여 재로그인을 합니다. <br>

접근하고자 하는 EC2 인스턴스에 SSM을 통해 접근해 보고, 다른 인스턴스에도 접근하여 테스트해 봅니다. <br>

로깅 설정을 했던 S3 버킷에서 로그를 확인합니다. <br>
![log1](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\log1.png) <br>

해당 로그를 다운로드해 확인해 보면 명령어 실행 이력과 결과를 확인할 수 있습니다. <br>
![log2](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\log2.png) <br><br>

## 마치며

Session Manager는 EC2 인스턴스에 접근할 때 추가 요금이 발생하지 않습니다. <br>
하지만 프라이빗 환경에서 사용하기 위해 생성한 인터페이스 엔드포인트가 시간당 0.013 달러이므로 3개의 엔드포인트를 생성하였으니 시간당 0.039 달러가 청구되긴 합니다. <br>
![price1](C:\Users\DohyeonLee\Desktop\0915컨퍼런스\블로그용\price1.png) <br>

사실 서버 접근통제를 위한 다양한 3rd Party 솔루션이 존재할 것입니다. <br>
하지만 AWS를 사용하는 고객이라면 Session Manager로 보다 간편하게 설정 및 관리하며, 비용 측면에서도 이점을 누리는 것이 어떨까요? <br><br>

긴 글 읽어주셔서 감사합니다. :smile: <br>