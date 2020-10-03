# 영역1 : 복원력을 갖춘 아키텍처 설계
## 안정적이고 복원력을 갖춘 스토리지 선택하기

### Instance Store 
### Elastic Block Store 
- 블록 스토리지
- 다양한 유형 : SSD / HDD
  - SSD PIOPS : 데이터베이스
  - HDD Cold : 장기간 싼 가격에 백업 데이터
- 암호화
- 스냅샷
- 프로비저닝된 용량 : 만약 16TB 이상 원할때 복수개를 만들어서 RAID로 구성
- EC2인스턴스와 다른 독립적인 수명주기
- RTO : 장애발생시 복구에 걸리는 최대시간
- RPO : 장애발생시 유실될 수 있는 데이터량
- 저장된 데이터는 가용영역 내에 자동복제됨
- 볼륨은 암호화가 가능 
- 데이터는 백업시 스냅샷으로 S3

### EFS
- 파일 스토리지, 공유스토리지
- 페타바이트 규모 파일 시스템
- 탄력적 용량
- NFS
- EC2에서 리눅스 기반 AMI에 마운트해서 사용 가능 
- EFS 아키텍쳐 

### S3
- 오브젝트 스토리지
- 스토리지 클래스 및 내구성 - 스탠다드, 스탠다드-IA
- 암호화(저장데이터) - SSE
- 암호화(전송데이터) - HTTPS 
- 버전관리 
- 액세스 제어
- 멀티파트 업로드 
- 인터넷/api 액세스 가능
- 사실상 무제한의 용량
- 리전 가용성
- 뛰어난 내구성 : 9'11%

### Glacier
- 데이터 백업 및 아카이브 스토리지
- 저장소 및 아카이브
- 검색 : 가속, 스탠다드, 벌크
- 암호화
- s3 정책

## 느슨한 결합을 구현하기 위한 서비스들
### 구성요소의 상태에서 결합 해제 - SQS
- 웹서버 -> 이메일서비스 -> 이메일서버
- 만약 이메일 서비스, 이메일서버 죽을 때 웹서버 데이터 없어진다(밀결합)
- 이 밀결합을 해제하기 위해 SQS를 쓴다. 가운데에 있는 SQS/LB가 강력한 내구성으로 데이터 보전해줌 

### 확장성을 위한 결합해제 - SQS, ELB
- 웹서버->로깅서비스->DynamoDB
- 웹서버->SQS->복수개의 로깅서비스 인스턴스 Autoscaling을 통해 분산할 수 있음.
- DynamoDB또한 복수개의 방식으로 도는 확장성을 가지고 있음

### 구성요소의 ID에서 결합해제 - EIP, Route53
- 기존 : 외부 클라이언트->VPC 퍼블릭IP
- 외부 클라이언트->탄력적(EIP)IP
- 외부 클라이언트->DNS

## 고가용성과 내결함성 
- 시스템이 더 느슨하게 결합될수록 더 쉽게 확장되며 내결함성도 강화할 수 있다
- 고가용성(High Availability) : 조금 느리더라도 작동 가능할때 (2개의 가용영역 4개 배포)
- 내결함성(Fault Tolerance) : 성능 SLA 또한 만족시켜야 (2개의 가용영역 8개 배포)
- Well Architected 프레임워크
* "단일 가용 영역"은 정답 아님
* 언젠가는 장애가 발생할 것으로 예상하고 이에 맞춰 설계

### CloudFormation
- AWS 리소스를 배포하기 위한 선언형 프로그래밍 언어 
- 리소스 집합을 스택으로 생성하여 배포 
- 템플릿은 리전별로 공통된 것을 사용, 리전마다 달라지는 내용은 매핑을 사용해서 meta data에 저장(ex AMI ID)
* AMI는 리전기반으로 저장되기 떄문에, AMI ID는 리전마다 다르다

### Lambda
- 이벤트 또는 시간 기반 간격에 대한 응답으로 상태 비저장 코드를 실행하는 완전관리형 컴퓨팅 서비스 
- EC2 인스턴스 및 AutoScaling 그룹과 같은 인프라를 관리하지 않고도 코드 실행 
- 실행할 때 사용한 자원에 대해서만 비용 청구 
* SSH 액세스 허용하지 않음, print 출력은 CloudWatch 로그 사용

# 영역2 : 성능이 뛰어난 아키텍처 정의

## 성능이 뛰어난 스토리지 및 데이터 베이스 선택

### EBS 볼륨 유형 
* SSD : 랜덤액세스
  - 범용 : 16TB/10000IOPS 네트워크 혼잡할 때 성능 떨어짐
  - 프로비저닝된 IOPS : 16TB/32000IOPS 네트워크에 할당된 대역폭과 EBS에 할당된 대역폭을 분리하여 네트워크 혼잡할때에도 성능 훌륭, 대표적으로 데이터베이스
* HDD : 백업 
  - 처리량 최적화 : 빠르게 백업, 빠르게 READ
  - 콜드 : 장기간 백업

### S3 버킷 
- 객체를 무한대로 저장하는 것이 가능
- 객체는 불변(단일 바이트를 변경하는 것이 객체를 바꿀 수 있는 유일한 방법)
- 객체가 AZ간에 복제된다
- 2가지 형태의 url 
  - 가상호스트기반 URL : bucket.s3-aws-region.amazonaws.com -> Route53으로 CNAME으로 DNS 설정 가능 
  - path형태
- **사용한 만큼만 비용을 지불**
  - 월별 GB, 리전 밖으로 송신하는 경우, PUT/COPY/POST/LIST/GET 요청에 대한 요금( 이런 요청이 잦을경우 elasticache, dynamoDB 병행해서 쓴다)
- 무료
  - 수신하는 것, 같은 리전의 CloudFront로 송신하는것 

### S3 스토리지 클래스 
- 일반목적 : **스탠다드** 
- 액세스빈도가 낮은 데이터 : **스탠다드 IA**
  - 저장된 GB당 비용이 저렴
  - PUT, COPY, POST, GET 요청당 비용이 높음
  - 최소 30일 스토리지
  - 백업파일, 로그파일

### S3 수명주기 정
- S3 수명 주기 정책을 이용해 생성 후 기간을 기준으로 객체의 스토리지 클래스를 변경/삭제
- 스탠다드에서 30일 이후 IA로 60일 이후에는 Glacier로 365일 이후에는 삭제하도록 설정할 수 있음 

### RDS의 적합한 usecase
- 복잡한 트랜잭션/쿼리
- 중간/높은 수준의 쿼리/쓰기 속도
- 단일 노드/샤드 사용
- 높은 내구성(Multi-AZ 사용)

### RDS가 적합하지 않은 usecase
- 대규모 읽기/쓰기 (DynamoDB 사용)
- 샤딩 (샤딩 클러스터 구축할 수 있는 EC2 Instance에 직접 데이터베이스 구축)
- 간단한 GET/PUT 요청 및 쿼리 
- RDBMS의 사용자 정의 사용하는 경우

### RDS 읽기 전용 복제본(Read Replica)
- MySQL, PostgreSQL, Aurora, mariaDB만 읽기전용 복제본이 지원됨 
- Select 쿼리에 대한 부하를 오프로드 분산 
- 최종적 일관성(eventually consistent)

### DynamoDB 프로비저닝된 처리용량
- 처리 용량 요구사항(읽기/쓰기)에 따라 리소스 할당 
- 읽기 용량 유닛(max 4kb)
  - 초당 1건의 **강력한 일관된** 읽기
  - 초당 2건의 **최종적 일관된** 읽기 
- 쓰기 용량 유닛(max 1kb)
  - 초당 1건의 쓰기(일관성 상관 없이)
 
## 캐싱을 적용하여 성능 개선
### CloudFront 
- 컨텐츠에 대한 캐싱 : 사용자가 콘텐츠를 요청 - CloudFront 엣지로케이션에 캐시된 객체 복사본 - 캐시되지 않은 콘텐츠가 오리진으로 검색됨
- 캐시에 저장하기 좋은 객체 : 세션상태, 장바구니, 제품카탈로그 (은행계정잔액 : 이 데이터는 정확성과 업데이트 여부가 매우 중요 critical data)
- 사용사례 및 이점
- 콘텐츠 : 정적 및 동적
- 오리진 : S3, EC2, ELB, HTTP 서버
- 프라이빗 콘텐츠 보호
- 향상된 보안 
  - AWS Shield 스탠다드 및 고급버전
  - AWS WAF

### ElastiCache
- 데이터베이스에 대한 캐싱 : 자주 쿼리되는 결과 캐싱, 쿼리에 대한 응답속도 개선 
- 지원하는 캐시 엔진 2가지
  - Memcached : 멀티스레딩, 유지관리용이, 수평적 조정(Scaling)
  - Redis : 데이터 구조 지원(복잡한 데이터/쿼리/소팅), 지속성(지속적으로 백업), Pub/Sub메시징(지정된 구독자에게 퍼블리싱),장애조치(읽기전용복제본)
  
## 탄력성과 확장성을 갖춘 솔루션 설계
### 수직적 조정과 수평적 조정 
* 수직적 조정(스케일업) : 인스턴스의 사양변경(micro->xlarge)
  * 장점) 멀티코어를 지원하는 경우 어플리케이션에 지장 없이 이용 가능 
  * 단점) 반드시 시간 지연이 있다 
* 수평적 조정(스케일아웃) : 인스턴스의 수 조정 

### 탄력성 구현 
- 로드밸런서에서 트래픽을 각각의 EC2로 분배 Auto Scaling정책이 이 EC2인스턴스들을 관리 
- ELB는 헬스체크를 통해 EC2 확인 후 트래픽을 보낸다 
- 임계값 및 시간 확인을 하는 CloudWatch 경보를 통해 Auto Scaling 트리거

### Auto Scaling
- 인스턴스 시장 또는 종료 
- 새 인스턴스를 로드 밸런서에 자동으로 등록
- multi AZ
- 시작 구성 : 완벽하게 구성된 인스턴스를 자동으로 시작하기 위해 사용하는 템플릿 
- 시나리오1 : 특정시간에 트래픽이 일시적으로 급증하는 경우(인스턴스 2개->8개) 
  - 최소 용량은 2개이지만 특정시간에 확장이 예정된 Auto Scaling 그룹 생성 
- 시나리오2 : 최적 인스턴스 9개, 반드시 6개 이상 실행해야하는 경우 
  - 3개의 가용영역에서 9개의 인스턴스 실행
  
### Auto Scaling 구성요소 
* 시작 구성 
  * EC2 인스턴스 크기 및 AMI 이름 지정 
* 그룹 
  * 시작 구성을 참조
  * 그룹의 최소/최대 원하는 크기를 지정 
    * 고가용성이 필요하다면 최소 2개 인스턴스를 다른 가용영역에 위치하도록 
    * 원하는 크기는 정책에 의해 수시로 변화함 
  * ELB 참조 가능
  * 상태 확인 유형 (ELB 헬스 체크)
* 정책
  * 스케일인/아웃 규모 지정
    * 지정된 시간에 스케줄에 의해 규모 지정도 가능 
  * 그룹에 여러개 연결해서 사용 가능 

### CloudWatch 지표 
- 모니터링 대상 : CPU, 네트워크 대기열 크기
- CloudWatch 로그 이해 : 로그를 한곳으로 모아서 관리하고, 특정 문자열에 대한 임계점 설정 후 알람 보내기/트리거 가능
- 기본지표와 사용자 지정 지표 간의 차이점 이해 
  * 사용자 지정 지표 : EC2 인스턴스 메모리 사용량

### Elastic Load Balancing
- 복수개의 EC2, AZ로 트래픽 분산 
- 시나리오 : 2개 가용영역에 6개 인스턴스, 데이터 티어는 MySQL, 가용성 높이려면?
  - Auto Scaling 그룹에서 인스턴스를 시작
  - 다중 AZ RDS로 데이터베이스를 마이그레이션 한다 

### 시험 tips 
- 데이터가 구조화되지 않은 경우 대용량의 경우 S3 (+빠른 쿼리 DynamoDB)
- 콘텐츠 캐싱에는 CloudFront 데이터베이스 캐싱에는 ElastiCache
- AutoScaling 사용해야하는 시기와 이유
- 워크로드와 성능요구에 적합한 인스턴스 및 데이터베이스 유형 선택


# 영역3 : 안전한 애플리케이션 및 아키텍쳐 지정

## 애플리케이션 보안 설계 

### 인프라
* 인프라 : 공동 책임 모델
* AWS 리소스 보호 
  - 최소 권한의 원칙 
  - 자격 증명 
* 시나리오 : 계정관리자가 퇴사할 때 AWS 인프라를 보호하기 위해 해야할 일 
  - 암호를 변경하고 루트 사용자에 MFA를 추가함
  - 키를 교체하고 IAM 사용자의 암호를 변경
  - 관리자의 IAM사용자를 삭제

### IAM
- 사용자와 권한을 중앙 제어 
- 사용자, 그룹 역할 및 정책을 생성 
- 액세스 제어 권한 정의 
- SAML 자격 증명 연동을 통해 외부 인증 연동 가능 

### 자격 증명
- IAM 사용자 : 계정 내에서 생성된 사용자
- EC2 인스턴스, Lambda 및 외부 사용자가 사용하는 임시 ID(Role, 토큰)
- 연동 : Active Directory ID 또는 기타 기업 자격 증명을 
- 웹 ID 연동 : STS 토큰을 발급해 역할 할당 

## 컴퓨팅/네트워크 아키텍처 보안 
### VPC
- 내부의 네트워크를 서브넷을 통해 조직화 
- 보안 : 보안그룹(EC2 방화벽)/액세스제어목록 NACL(서브넷 방화벽)
- 네트워크 격리 : 인터넷 게이트웨이/NAT/VPC 게이트 웨이
- 트래픽 방향 : 라우팅 테이블 

### 서브넷 사용 방법
- 권장 사항 : 서브넷을 사용하여 인터넷 접근성 정의 
- 퍼블릭 : 인터넷에 대한 인바운드/아웃바운드 액세스를 지원하려면 인터넷 게이트웨이에 라우팅 테이블 항목을 포함(DB 리소스 절대 안됨)
- 프라이빅 : 일반적으로 NAT를 향해 라우팅. 퍼블릭 인터넷에서 직접 액세스 불가. 제한된 범위의 아웃바운드 전용(NAT를 향해)

### 보안그룹과 네트워크 ACL 비교 
보안그룹 인스턴스 내부에서 연결해서  Elastic Network Interface에 적용 
ACL 상태 비저장 (아웃바운드가 허용되었다고 해더라도 인바운드는 별도의 룰을 통해 제어)

### 보안그룹 
- 리소스에서 송수신되는 트래픽을 제어 
- 데이터 티어 보안그룹/애플리케이션 티어 체이닝 : 보안그룹 인바운드 외부로부터 접속 요청 허용하지 않도록 설정 가능
  - 애플리케이션 티어의 각 인스턴스를 웹 티어 보안그룹에서 인바운드 HTTP 트래픽을 허용하는 보안 그룹에 연결 
  - NACL은 안됨 : 상태 비저장; 다른 애플리케이션 티어 인스턴스에서 액세스가 허용 

### VPC 연결 
- 트래픽을 가져오는 서비스를 파악
- 인터넷 게이트웨이 : 인터넷에 연결
- 가상 프라이빅 게이트웨이 : VPN에 연결, 퍼블릭 인터넷 액세스에 관여하지 않음 
- AWS 직접 연결 : 전용 파이프
- VPC 피어링 : 다른 VPC에 연결, 다른 리전과도 연결 가능  
- NAT 게이트웨이 : 프라이빗 서브넷에서 아웃바운드 인터넷 트래픽 허용 
- 인터넷이 안될 떄 
  - VPC에 인터넷 게이트웨이가 포함되어 있고, 서브넷의 라우팅 테이블이 0.0.0.0/0을 인터넷 게이트웨이로 라우팅하고 있는지 확인
  - 보안그룹이 포트 80에서 인바운드 액세스를 허용하는지 확인
  - 네트워크 ACL이 포트 80에서 인바운드 액세스를 허용하는지 확인(하지만 늘 열려있지 stateless) 

### 프라이빗 인스턴스의 아웃바운드 트래픽 
- NAT 인스턴스 : 포트 포워딩
- NAT 게이트웨이 : 자체적으로 확장 가능, 장애 발생시 자동 failover 

## 데이터 보호 
### 전송 데이터 
- AWS 인프라 안과 밖에서 데이터 전송 
  - 웹을 통한 SSL
### 저장데이터 
- S3에 저장된 데이터는 기본적으로 프라이빗이고 액세스하려면 자격증명 필요 
  - HTTP 액세스
  - 모든 객체에 대한 액세스 감사
  - ACL 및 정책 지원 : 버킷, 접두사(디렉터리/폴더), 객체
- 서버 측 암호화 옵션 : 데이터를 받는 쪽에서 데이터를 암호화
  - S3가 관리하는 키 (SSE-S3)
  - KMS가 관리하는 키(SSE-KMS)
  - 고객 제공 키(SSE-C)
- 클라이언트 측 암호화 옵션 : S3로 보내기 전에 데이터 암호화
  - KMS에 저장된 CMK 사용(고객 마스터키를 사용하여 클라이언트가 먼저 객체 데이터를 암호화하는데 사용할 수 있는 CMK 요청)
  - 애플리케이션 내에 저장된 마스터키 사용
- 시나리오 : 어플리케이션은 EC2인스턴스에서 실행되는데 이 애플리케이션이 버킷에 안전하게 액세스하도록 하려면?
  - S3의 버킷에 대한 액세스 권한을 부여하는 정책을 가진 **IAM 역할**을 EC2인스턴스에 연결

### 키 관리 
- Key Management Service
  - 고객 소프트웨어 기반 키 관리
  - 다른 AWS 서비스와 통합 : EBS, S3, RDS, Redshift, Elastic Transcoder, WorkMail, EMR (문제에는 최근 반영분은 업데이트가 안되어있음)
  - 애플리케이션 직접 사용(데이터베이스 로그인 암호를 KMS를 통해 암호화된 정보로 접근하도록 함)
- AWS CloudHSM
  - 하드웨어 기반 키 관리 
  - FIPS 140-2 준수(금융권)
  
### 시험 팁
- 루트 사용자는 사용 금지
- 보안 그룹은 allow만. 네트워크 ACL은 allow/deny 모두 사용
- 액세스 키보다 IAM역할을 선호 

# 영역4 : 비용에 최적화된 아키텍처 설계

## 비용에 최적화된 스토리지를 설계하는 방법 결정

### AWS 요금 개요 
* 사용한 만큼 
* 예약한 경우 지불 비용 감소
* 더 많이 사용할수록 단위당 더 적은 비용 지불 

### 요금이 부과되는 3가지 요소
* 컴퓨팅
* 스토리지
* 데이터 전송(아웃바운드)

### S3 스토리지 클래스 
* 스탠다드
* 스탠다드-IA
  - 보관비용은 저렴하지만 액세스 비용이 비쌈
* Glacier 
  - 보관비용은 가장 저렴하지만 리트리브 시 3~5시간 걸림, 웹에 공개 불가 

### EBS 사용 비용 추정 
- 볼륨
- IOPS(초당 입출력 작업 수)
- 스냅샷
- 데이터 전송 

### CloudFront 비용 이점 
- s3와 cloudfront간의 데이터 전송에 대해 요금이 부과되지 않음 
- EC2 인스턴스에 대한 컴퓨팅 워크로드를 줄이는 데 사용 가능 
- 비용 추정 고려사항 
  - 트래픽 배포 
  - 요청 
  - 데이터 송신

## 비용에 최적화된 컴퓨팅을 설계하는 방법 결정
### EC2 요금 추정 시 고려 사항 
- 서버사용시간
- 머신 구성
- 머신 구입 유형 : 예약? 스팟?
- 인스턴스 수 : 고정인가?
- 로드밸런싱
- 세부 모니터링
- Auto Scaling 
- 탄력적 IP주소
- 운영체제 및 소프트웨어 패키지

### EC2 요금 요소
- 인스턴스 패밀리
- 테넌시  dedicate? 공용?
- 요금 옵션 : 예약? 스팟?
* 인스턴스 스토리지는 무료지만 임시성/휘발성

### EC2 비용 절약 방법
- 예약인스턴스(RI)
  - 온디맨드 가격에 비해 상당히 할인된 가격 
  - RI유형 : 스탠다드 RI, 변환가능한 RI, 예정된 RI
  - 1~3년동안 꾸준히 사용하는 경우 
- 스팟인스턴스
  - 온디맨드 가격에 비해 대폭 할인된 가격 
  - 단기간, 중간에 실패하더라도 리트라이(연기) 가능한 작업 (맵리듀스, 배치작업)
- 예정된 인스턴스
  - 온디맨드 가격에 비해 5~15퍼정도 
  - 특정 시간에 작업을 실행하는 경우 
  
### 서버리스 아키텍쳐
- 서버리스아키텍처는 컴퓨팅 워크로드를 줄일 수 있다 
- 특히 Lambda 

### 시험 팁
- 미래를 예측할 수 있다면 그렇게 해야한다. 추정 및 확인 필요 
- 사용되지 않은 CPU 시간은 비용 낭비(EC2 인스턴스 낭비)
- 가장 비용 효과적인 데이터 스토리지 서비스 및 클래스 사용 
- 각 워크로드에 대해 가장 비용 효과적인 EC2 

  
# 영역5 : 운영에 탁월한 아키텍처 정의

## 솔루션에서 운영 우수성을 지원할 수 있는 설계 기능 선택 

### 운영 우수성 : 설계 원칙 
- 코드로 운영 수행
- 문서에 주석 추가(버전관리)
- 빈번하고 작은 규모로 되돌릴 수 있는 변화 수행 
- 운영 정차를 빈번하게 재정의
- 실패 예상
- 모든 운영 실패에서 학습(피드백루프)

### 운영 우수성 제공 서비스
- Config : 환경 스냅샷 생성, 환경 변경 기록, 변경에 대한 영향 추적
- CloudFormation : 템플릿으로 환경 
- Trusted Advisor : 사용 패턴 분석 및 알람
- Inspector : EC2 인스턴스 내부에 에이전트 설치 후 보안 
- VPC Flow : VPC 내부의 네트워크에 대해 이벤트 중심으로 추적으로 트러블 슈팅/보안 
- CloudTrail : API 단위로 누가 명령어를 보냈는지

### CloudWatch 
- 통계치 분석 수집 경보 알람 
- 시나리오 : CPU 사용량을 5분단위로 3번의 기간동안 70% 도달할경우 경보 발생하게 함 -> 10분동안 80% 도달한다면 -> 경보는 0개 발생 
- CloudWatch 지표는 HTTP 404 오류를 수집하지 않는다
- CloudWatch 로그를 사용하여 EC2 인스턴스에서 웹 서버 로그를 가져오면, HTTP 404 오류를 수집할 수 있다

### 시험 팁
- IAM 역할은 키와 암호보다 손쉽고 안전
- 시스템 전반에서 지표를 모니터링 CloudWatch 
- 해당되는 경우 지표에 자동 조치(404, cpu사용률 -> CloudWatch Trigger 또는 Alarm)
- 변칙적인 상황에 대해 경보 발행 CloudWatch 경보