# VPC에 대해 알아보기

 VPC는 Virtual Private Cloud의 약자로 AWS 클라우드 상에서 논리적으로 격리된 네트워크 공간을 할당하여 가상 네트워크를 구성할 수 있는 서비스입니다. VPC에 대해 이해하기 위한 핵심 개념은 아래와 같습니다.

-   VPC
-   서브넷 (Subnet)
-   Private IP, Public IP, Elastic IP
-   라우트 테이블 (Route Table)
-   네트워크 ACL (Network ACL)
-   보안 그룹 (Security Group)
-   인터넷 게이트웨이 (Internet Gateway)

 이제 각 개념들에 대해 살펴보겠습니다.

## VPC

 VPC는 포스트의 제일 위에서 설명했듯이 논리적으로 격리된 가상 네트워크를 말합니다. VPC는 하나의 리전 내에서만 속할 수 있습니다. 즉, 여러 리전에 걸쳐서 구성이 불가능합니다. VPC가 하나의 리전 내에서만 속할 수 있지만, 리전 내에 여러 개의 Available Zone에 걸쳐서 생성될 수 있습니다. VPC를 구성할 때는 IPv4 주소 범위를 CIDR(Classless Inter-Domain Routing) 블록 형태로 지정해야 합니다.

![VPC](./images/vpc.png)

 여기서 CIDR 블록은 IP의 범위를 지정하는 방식입니다. CIDR 블록은 아이피 주소와 슬래쉬('/') 뒤에 따라오는 넷마스크 숫자로 구성되어 있습니다. 위 그림에서 보이는 "10.0.0.0/16" 과 같은 형식이 바로 CIDR 블록입니다. **허용된 블록 크기는 /16(65,536개의 IP 주소) ~ /28(16개의 IP주소)**입니다. 위 아이피 주소의 넷마스크는 16으로 되어 있는데, 아이피 주소를 2진법으로 변환했을때 앞에서부터 16비트까지가 네트워크 주소임을 나타내는 것입니다.

![CIDR](./images/cidr.png)

 VPC를 생성할 때 CIDR 범위를 지정하는데 특별한 제약은 없지만 인터넷과 연결되는 경우 주의해야할 점이 있습니다. 예를 들어 CIDR 블록을 52.12.0.0/16 으로 지정했다고 가정해보겠습니다. 생성된 VPC 내에서는 52.12.0.0/16으로 접속되는 트래픽은 VPC 내부로 라우팅 되겠지만 이 범위는 인터넷 망에서도 사용할 수 있는 IP 대역입니다. 따라서 VPC에서는 52.12.0.0/16 대역에 속한 인터넷 IP 대역에 접속하는 것이 불가능해집니다. 따라서 **인터넷 연결이 필요한 경우 반드시 사설망 대역을 이용해야 하며, 인터넷 연결이 필요하지 않더라도 사설망 대역을 이용하는 것을 권장**합니다.

 VPC의 사설망 대역은 아래와 같습니다.

-   10.0.0.0/8
-   172.16.0.0/12
-   192.168.0.0/16

## 서브넷 (Subnet)

 서브넷은 VPC 내에서 IPv4 주소가 CIDR 블록에 의해 나눠진 주소 단위입니다. 서브넷은 물리적으로 가용 공간인 Available Zone(이하 AZ)과 연결됩니다. VPC는 N개의 서브넷을 가지게 되고, 서브넷은 하나의 AZ에 연결됩니다. 각 서브넷 블록에서 첫 4개의 주소와 IP의 마지막 주소는 사용할 수 없으므로 인스턴스에 할당할 수 없습니다. 예를 들어 192.168.0.10/16 의 주소는 아래와 같이 5개의 주소에 대해 미리 예약되어 있습니다.

-   192.168.0.0 : 네트워크 주소입니다.
-   192.168.0.1 : AWS에서 VPC 라우터용으로 예약한 주소입니다.
-   192.168.0.2 : AWS에서 DNS의 기본 IP 주소는 기본 VPC 네트워크 범위에 2를 더한 주소입니다.
-   192.168.0.3 : AWS에서 앞으로 사용하려고 예약한 주소입니다.
-   192.168.0.255: 네트워크 브로드 캐스트 주소입니다.

![Subnet](./images/subnet.png)

 위 그림은 VPC의 서브넷에 대한 대략적인 그림입니다. 앞서 말씀드린 것처럼 하나의 VPC에는 N개의 서브넷이 존재하고, 각 서브넷은 하나의 AZ에 속하게 됩니다. 이 서브넷들은 라우팅 테이블에 의해 연결될 수 있습니다. 즉, 여러 AZ에서 리소스들이 물리적으로 다른 위치에 있더라도 하나의 네트워크에 속해있는 것처럼 가상의 네트워크 망을 구축할 수 있게 되는 것입니다.

---

## 참고자료

[https://docs.aws.amazon.com/ko\_kr/vpc/latest/userguide/what-is-amazon-vpc.html](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html)

[https://www.44bits.io/ko/post/understanding\_aws\_vpc](https://www.44bits.io/ko/post/understanding_aws_vpc)

[https://bluese05.tistory.com/45?category=559701](https://bluese05.tistory.com/45?category=559701)

[https://docs.aws.amazon.com/ko\_kr/vpc/latest/userguide/VPC\_ElasticNetworkInterfaces.html](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html)

[https://bcho.tistory.com/779](https://bcho.tistory.com/779)

[https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098](https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098)
