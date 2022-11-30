---
title: 폐쇄망에서 AWS Elastic Beanstalk으로 WordPress 웹 사이트 구축하기
authors:
  - juyeon-lee
date: 2022-11-30T13:22:45.862Z
feature_image: images/cover.png
categories:
  - Hands On
---
안녕하세요! 클라우드메이트 이주연입니다.

오늘 저는 AWS 환경을 폐쇄망으로 하여 Elastic Beanstalk으로 Wordpress 웹사이트를 구축하는 핸즈온을 준비했습니다.

# 들어가기 전에

먼저 기업에서 사용하는 클라우드에 대해 알아보겠습니다.

*\[ 퍼블릭 클라우드 / 프라이빗 클라우드 / 하이브리드 클라우드 / 멀티 클라우드 ]*

### 하이브리드 클라우드

하이브리드 클라우드는 기존의 온프레미스 환경이나 프라이빗 클라우드에 퍼블릭 클라우드를 같이 사용하는 형태입니다.

온프레미스나 프라이빗 클라우드가 가지는 보안성과
퍼블릭 클라우드의 유연성 등 둘의 장점을 동시에 가질 수 있습니다.

### 멀티 클라우드

멀티 클라우드는 서로 다른 퍼블릭 클라우드를 함께 사용하는 형태입니다.

하나의 클라우드 서비스에 장애가 생기게 되더라도 서비스가 중단되지 않게 대비하여 고가용성을 확보할 수 있습니다.

## Cloud + VPN

하이브리드 클라우드나 멀티 클라우드 구성 시 VPN을 사용하여 프라이빗 한 네트워크 연결을 할 수 있습니다.
또한 온프레미스와 클라우드를 VPN 연결하여 사용할 수도 있습니다.

이렇게 클라우드를 연결해서 쓰는 이유는 무엇일까요?

Cloud가 가진 탄력성, 신속성으로 필요할 때만 컴퓨팅 리소스를 생성하여 사용하고, 사용한 만큼만 비용을 지불하여 장비를 사는 등의 비용을 절감시킬 수 있는 장점이 있습니다. 또 글로벌한 회사의 경우, 해외에 데이터 센터를 구축하지 않아도 손쉽게 해외 Region에 컴퓨팅 리소스를 생성할 수 있습니다.

## 폐쇄망

그렇다면 폐쇄망이란 무엇일까요? 폐쇄망이란 인터넷이 되지 않는 환경을 말합니다.

보안이 중요시되는 금융권 등에서는 정보 유출과 외부 침입을 막기 위해 업무용 폐쇄망과 인터넷망을 분리합니다. 폐쇄망에서는 인터넷이 통하지 않기 때문에 데이터가 외부로 노출되는 위험을 줄일 수 있습니다.

## VPC Endpoint

인터넷이 되지 않는 환경에서 AWS 서비스를 사용하기 위해서는 VPC Endpoint를 생성해야 합니다.

VPC Endpoint는 인터넷 연결 없이 AWS 서비스에 프라이빗하게 연결할 수 있는 VPC의 진입점입니다. 트래픽이 AWS 외부로 나가지 않고 프라이빗하게 다른 AWS 서비스에 연결할 수 있기 때문에 좀 더 빠르고, 보안에 강하다는 장점이 있습니다.

VPC Enpoint에는 두 가지 종류가 있습니다.

![](endpoint.png)

* Interface Endpoint\
  \
  Interface Endpoint는 Private Subnet 안에 위치하고, Private IP를 이용하여 다른 서비스로 연결시켜줍니다.
  현재 많은 종류의 AWS 서비스들을 지원하고 있습니다.\
  \
  EC2에서 Private Subnet 내부의 Interface Endpoint를 통해 AWS 서비스에 액세스하게 됩니다.
* Gateway Endpoint\
  \
  Gateway Endpoint는 라우팅 테이블을 이용하여 다른 서비스에 연결합니다.
  Gateway Endpoint 생성 시 라우팅 테이블에 자동으로 Gateway Endpoint가 추가됩니다.
  현재 Gateway Endpoint가 지원하는 서비스에는 S3와 DynamoDB가 있습니다.\
  \
  EC2에서 라우터를 거쳐 Gateway Endpoint를 통해 S3에 엑세스하게 됩니다. 

# Hands-On

## 아키텍처

![](architecture_with_gcp.png)

핸즈온을 진행하기 전 다른 클라우드 환경과 VPN 연결이 필요합니다.
저는 미리 AWS와 GCP를 VPN 연결해 두었습니다.
VPN 연결에 관한 더 자세한 내용은 이전 글을 참고하여 설정해 주시기 바랍니다.

<https://tech.cloudmt.co.kr/2022/09/30/%EC%9D%B8%ED%94%84%EB%9D%BC-%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%9D%BC%EB%A9%B4-%EA%BC%AD-%EC%95%8C%EC%95%84%EC%95%BC%ED%95%98%EB%8A%94-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-vpn/>

AWS 환경에는 Internet Gateway나 NAT Gateway를 만들지 않았고, GCP 환경에서는 인터넷이 가능한 퍼블릭 환경으로 생성하였습니다.
그리고 GCP에서 AWS에 생성한 Wordpress 웹 사이트에 접근하기 위해 Windows VM 인스턴스를 생성해 두었습니다.

이제 Beanstalk으로 웹 애플리케이션을 배포하고, WordPress를 설치하여 RDS와 연결할 것입니다.
또한 ELB의 액세스 로그를 S3에 저장하고, 관리자가 EC2 인스턴스에 접근할 수 있도록 Systems Manager를 사용할 수 있도록 하려고 합니다.

## VPC, Subnet, 라우팅 테이블 생성

먼저 서울 리전에 VPC를 생성하고, 고가용성을 위해 두 개의 AZ에 EC2 인스턴스와 RDS를 위한 Subnet을 생성합니다. (2a-WEB, 2c-WEB, 2a-DB, 2c-DB)

![](vpc_subnet.png)

생성한 서브넷을 라우팅 테이블에 연결합니다.

![](route_table1.png)

## VPC Endpoint 생성

Private Subnet이기 때문에 다른 AWS 서비스에 액세스 하기 위해 VPC Endpoint를 생성해야 합니다.
Elastic Beanstalk을 사용하기 위해 다음의 VPC Endpoint들을 생성합니다. 

![](vpc_endpoint.png)

참고 : <https://aws.amazon.com/ko/premiumsupport/knowledge-center/elastic-beanstalk-instance-failure/>

Interface Endpoint 생성 시 이전에 생성한 VPC, Subnet, 보안 그룹을 선택합니다.
Gateway Endpoint 생성 시에는 VPC와 라우팅 테이블을 선택하고, 생성이 되면 지정한 라우팅 테이블에 자동으로 Gateway Endpoint가 추가됩니다.

![](added_endpoint.png)

## SSM

Beanstalk에서 생성되는 EC2 인스턴스에서 AWS Systems Manager(SSM)를 사용하기 위해, SSM과 관련된 Endpoint를 추가로 생성합니다.
-> 선택한 보안 그룹 인바운드 규칙에 443(HTTPS) 추가

![](ssm_endpoint.png)

Beanstalk 생성 시 가상 머신 권한에 기본적으로 적용되어 있는 aws-elasticbeanstalk-ec2-role 역할에는 5개의 정책이 적용되어 있습니다.
IAM에서 SSM을 사용하기 위한 새로운 역할을 생성하여  5개 정책과 AmazonSSMManagedInstanceCore 정책을 추가해 줍니다.

![](ssm_role.png)

SSM에 관한 더 자세한 내용은 이전 글을 참고해 주시기 바랍니다.

<https://tech.cloudmt.co.kr/2022/09/29/aws-systems-manager%EC%9D%98-session-manager%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-%ED%94%84%EB%9D%BC%EC%9D%B4%EB%B9%97-%ED%99%98%EA%B2%BD%EC%9D%98-%EC%84%9C%EB%B2%84-%EC%A0%91%EA%B7%BC%ED%86%B5%EC%A0%9C-%EA%B5%AC%ED%98%84/>

## Elastic Beanstalk 생성

Wordpress 웹사이트를 구축하기 위해 Beanstalk에서 웹 서버 환경을 생성합니다.

애플리케이션 이름, 환경 이름과 도메인을 입력하고,
플랫폼으로는 PHP를 선택합니다.

![](beanstalk.png)

추가 옵션 구성을 클릭하여 다음과 같이 설정합니다.

용량 

* 환경 유형 : 로드 밸런싱 수행

![](beanstalk_yongryang.png)

네트워크 

* 로드 밸런서 설정 : 내부
* VPC, 로드 밸런서 서브넷(2a-WEB, 2c-WEB), 인스턴스 서브넷(2a-WEB, 2c-WEB), 데이터베이스 서브넷(2a-DB, 2c-DB) 선택

![](beanstalk_network.png)

인스턴스

* 보안 그룹 : Interface Endpoint 생성 시 선택한 보안 그룹을 선택

![](beanstalk_instance.png)

로드밸런서

* 로그 저장 : 활성
* S3 버킷 선택

![](beanstalk_lb.png)

보안

* IAM 인스턴스 프로파일 : 위에서 생성해둔 역할 선택

![](beanstalk_security.png)

환경을 생성합니다.

![](beanstalk_green.jpg)

## Beanstalk url 접속 확인

생성된 Beanstalk 환경의 URL로 들어가서 페이지가 잘 뜨는지 확인해 봅시다.

![](error_page.png)

AWS 환경을 폐쇄망으로 구성했기 때문에 외부에서는 접속할 수 없는 것을 확인할 수 있습니다.

이번에는 GCP에 생성해 놓은 Windows VM 인스턴스에 RDP 접속을 해서 해당 URL에 들어가 봅니다.
PHP 샘플 코드 화면이 잘 뜨는 것을 볼 수 있습니다.

![](test_screen.png)

## SSM 접속 확인

EC2에 가서 Beanstalk으로 생성된 EC2 인스턴스에서 SSM 접속이 잘 되는지 확인해 봅시다.
SSM의 연결 버튼이 활성화되어 있고, SSM 화면에 잘 접속되는 것을 확인할 수 있습니다.

![](ec2_ssm.png)

![](ssm_screen.png)

## RDS에서 MySQL 생성

WordPress에 연결할 RDS 데이터베이스를 생성합니다.

* 엔진 유형 : MySQL
* DB 클러스터 식별자 : wordpress
* 마스터 사용자 이름 : admin
* 마스터 암호 : qwer1234
* VPC, 보안 그룹 선택 -> 보안 그룹 인바운드 규칙에 3306(MYSQL/Aurora) 추가

  ![](rds.png)

생성된 데이터베이스의 엔드포인트를 복사합니다.

## CodeCommit에 WordPress 파일 업로드

WordPress 사이트에서 파일을 다운로드합니다.
https://ko.wordpress.org/download/

압축을 풀고 wp-config-sample.php 파일을 복사하여 wp-config.php 파일을 생성합니다.
wp-config.php 파일을 다음과 같이 수정합니다.

```
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'admin' );

/** Database password */
define( 'DB_PASSWORD', 'qwer1234' );

/** Database hostname */
define( 'DB_HOST', '<데이터베이스 엔드포인트>' );
```

CodeCommit에서 리포지토리를 생성하고, WordPress 파일을 업로드합니다.
![](codecommit.png)

## CodePipeline으로 WordPress 코드 배포

CodePipeline에서 새 파이프라인을 생성합니다.

* 파이프라인 이름 입력
* 소스 공급자 : CodeCommit
* 리포지토리 이름 선택
* 브랜치 이름 선택
* 빌드 스테이지 건너뛰기
* 배포 공급자 : AWS Elastic Beasntalk
* 애플리케이션 이름, 환경 이름 선택

![](codepipeline.png)

파이프라인이 생성되면 CodeCommit에 업로드된 파일들이 Beanstalk으로 배포됩니다.
이제 CodeCommit의 파일이 업로드되면 CodePipeline을 통해 자동으로 Beanstalk에 배포됩니다.

## WordPress 배포 확인

GCP의 Windows VM 인스턴스에 접속하여 Beanstalk url로 들어가 WordPress가 잘 배포되었는지 확인합니다.

![](wordpress_main_page.png)

## 마지막으로

긴 글 마지막까지 읽어주셔서 감사합니다.

좋은 하루 보내세요 :)