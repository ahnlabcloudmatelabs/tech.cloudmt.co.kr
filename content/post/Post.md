---
author: sejun kim
date: "2019-08-09T00:00:00Z"
tags:
- AWS
- Azure
- VPN
- VPC
- vNet
- Peering
- Terraform
- VGW
- HOL
- Hands on Lab
title: Azure와 AWS간 VPN 연결하기 (Terraform)
---

Azure Virtual Network와 AWS VPC(Virtual Private Cloud)를 논리적으로 연동하고 싶은 분들이 계실 것입니다. Azure를 사용하면서 AWS도 함께 사용하고 싶고, 각각 Cloud 업체의 장점을 활용하고 싶을 수도 있습니다. 그러한 경우 두 개의 Network를 논리적으로 연결하고 싶은 욕망이 생길 것이고 이를 관리하길 원하실 것입니다.

그러한 요구사항을 해소하기 위해 Azure Virtual Network Gateway와 AWS VPN Connection을 연결하여 Azure Virtual Network와 AWS VPC를 논리적으로 연결하는 방법에 대해 알아봅시다.

![Azure_to_AWS_VPN](https://techblogst.blob.core.windows.net/img/2019-08-09-Terraform/Azure_to_AWS_VPN.PNG)

## Azure Virtual Network Gateway

Azure Virtual Network Gateway(Network GW)는 IPSec VPN(Site-to-Site)과 SSL VPN(Point to Site)을 모두 지원합니다. 이 두 가지를 전부 구현해 주면서 관리형 서비스로 Azure에서 직접 관리하기 때문에 사용자는 리소스 모니터링만 하면 됩니다. Network GW는 여러가지를 지원하지만, SKU(성능)에 따라 지원하는 대역폭과 기능이 조금씩 다릅니다.

| SKU | 가격 | Bandwidth | S2S | P2S | OpenSSL 지원 |
|---|---|---|---|---|---|
| Basic | ₩40.49(대략) /시간 | 100 Mbps | 최대 10 | 최대 128 | No |
| VpnGw1 | ₩213.6835(대략) /시간 | 650 Mbps | 최대 30* | 최대 250** | Yes |
| VpnGw2 | ₩551.0785(대략) /시간 | 1 Gbps | 최대 30* | 최대 500** | Yes |
| VpnGw3 | ₩1,405.8125(대략) /시간 | 1.25 Gbps | 최대 30* | 최대 1,000** | Yes |

\*11~30는 별도 과금

\**129 이상은 별도 과금

[**참고 링크**](https://azure.microsoft.com/ko-kr/pricing/details/vpn-gateway/)

Network GW는 기본적으로 Active-Standby 구조를 가지며, 1대를 이용해도 이중화를 지원한다. 만약 Active-Active mode를 사용할 경우 Active-Active mode를 활성화 하면 되며 물리적으로 2개의 Instance가 생성되고 과금 역시 2배가 됩니다.

SSL VPN은 인증서를 직접 업로드하면 연결이 가능하며 OpenSSL, IKEv2, PPTP를 이용할 수 있습니다.

Network GW는 Connection이라는 것을 이용하여 VPN연결을 할 수 있으며 송신/수신을 담당합니다. connection을 만드려면 반대쪽 VPN장비의 Public IP를 이용하여 Local Network Gateway를 생성해야 합니다.

Network GW를 이용하여 Virtual Network(vNet)끼리 연결할 수 있으며, BGP Route를 이용하면 [**vNet을 거쳐 다른 vNet과 통신을 지원**](https://azure.microsoft.com/ko-kr/pricing/details/vpn-gateway/)합니다.

## AWS Virtual Gateway

AWS Virtual Gateway(VGW)는 하나의 VPC와 연결할 수 있습니다. 이 VGW는 하나 이상의 VPN Connection을 가질 수 있으며, 최대 1.25Gbps 대역폭을 지원합니다. SSL VPN(Point to Site)은 지원하지 않으며 별도의 성능을 보장하기 위한 정책은 없습니다. (SSL VPN은 별도의 서비스를 구성하셔야 합니다.)

[**참고 링크**](https://aws.amazon.com/ko/vpn/pricing/)

AWS VPN Connction은 기본적으로 2대의 Instance를 지원하고 있으며 반대쪽 VPN 장비의 설정에 따라 Acrive-Standby 또는 Active-Acrive mode를 사용할 수 있습니다.

AWS VPN Connction을 만드려면 반대쪽 VPN장비의 Public IP를 이용하여 Customer Gateway를 생성해야 하며, 과금은 VPN Connection 수 만큼 과금됩니다. 또한 AWS VPN Connction의 경우 수신측만 담당하고 있으며 VPN 장비에 DPD 설정이 되어있어야 합니다. 이유는 AWS VPN Connection의 경우 Tunnel에 Traffic이 10초간 흐르지 않을 셩우 Tunnel Down을 실행하기 때문에 DPD 설정이 되지 않는 장비라면 주기적으로 Tunnel에 트래픽을 흘려보내주는 설정을 해야 합니다. [**참고 링크**](https://aws.amazon.com/ko/premiumsupport/knowledge-center/vpn-tunnel-instability-inactivity/)

## Azure to AWS VPN Terraform

Azure와 AWS간 VPN연결을 하려면 Azure Portal과 AWS Console을 계속 왔다갔다 하면서 설정을 확인해야 합니다. 제가 몇번 해보니 너무 귀찮아서 배포자동화를 생각해 봤고 두 개의 Cloud업체에 Plugin을 제공하는 Terraform을 이용하여 구성해 봤습니다.

[**여기**](https://github.com/kimsejun2000/Azure_to_AWS_VPN)에 자세한 설명을 해놨으니 참고하여 설정하면 좋을 것 같습니다.
