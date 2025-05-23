# 🏦 RDS, Aurora & ElastiCache

## 🔍 Amazon RDS 개요

RDS는 Relational Database Service의 약자이다

이것은 쿼리 언어로 SQL을 사용하는 DB용 관리형 데이터베이스 서비스이다.

AWS가 관리하는 클라우드에서 데이터베이스를 생성할 수 있게 해준다

지원하는 데이터베이스는:

- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- Aurora (AWS 독점 데이터베이스)

## 💪 EC2에 DB를 배포하는 것보다 RDS를 사용하는 장점

RDS는 **관리형 서비스**로 단순한 데이터베이스 기능 이외에도 아래 기능들을 한다:

- 자동 프로비저닝, OS 패치
- 지속적인 백업 및 특정 시점 복원 (Point in Time Restore)!
- 모니터링 대시보드
- 향상된 읽기 성능을 위한 읽기 복제본
- DR(재해 복구)을 위한 다중 AZ 설정
- 업그레이드를 위한 유지 관리 윈도우
- 확장 기능 (수직 및 수평)
- EBS로 백업되는 스토리지

**그러나** 인스턴스에 SSH로 접속할 수는 없다. 

## 💽 RDS - 스토리지 자동 확장(시험에 나올 수 있음)

RDS 데이터베이스 인스턴스의 스토리지를 동적으로 늘릴 수 있다.

RDS가 무료 데이터베이스 스토리지가 부족한 것을 감지하면 자동으로 확장한다

**최대 스토리지 임계값**을 초기에 설정해두어야 한다.(DB 스토리지의 최대 제한)

다음 조건에서 자동으로 스토리지를 수정한다:

- 무료 스토리지가 할당된 스토리지의 10% 미만일 때
- 낮은 스토리지 상태가 최소 5분 동안 지속될 때
- 마지막 수정 이후 6시간이 경과했을 때

**예측할 수 없는 워크로드**가 있는 애플리케이션에 유용한 것이다.

모든 RDS 데이터베이스 엔진을 지원한다. 

## 📚 읽기 확장성을 위한 RDS 읽기 복제본

RDS 읽기 복제본을 사용하면 읽기 작업을 여러 데이터베이스 인스턴스로 분산할 수 있다.
![[스크린샷 2025-05-04 오후 4.03.49.png]]
 

- 최대 15개의 읽기 복제본
- AZ 내, 교차 AZ 또는 교차 리전에 생성 가능
- 읽기의 복제는 **비동기식**으로 진행되며 읽기 작업은  일관성을 갖는다. 
- 복제본을 자체 DB로 승격시킬 수 있다. 이때 메인이 되기 위한 설정은 해주어야 한다. 
- 애플리케이션은 읽기 복제본을 사용하기 위해서는 연결을 업데이트해야 한다.

## 📝 RDS 읽기 복제본 - 사용 사례
![[스크린샷 2025-05-04 오후 4.07.48.png]]
일반적인 부하를 받는 프로덕션 데이터베이스가 있다 (노란색)

그런데 일부 분석을 실행하기 위해 report 애플리케이션을 실행하려고 한다고 하자

이 때 메인 데이터베이스에 영향을 주지 않기 위해 읽기 복제본을 생성하는 것이다

읽기 복제본은 SELECT(=읽기) 유형의 명령문에만 사용된다는 것을 유의해자(INSERT, UPDATE, DELETE는 안 됨)

## 💸 RDS 읽기 복제본 - 네트워크 비용

AWS에서는 데이터가 다른 리전으로 이동할 때 비용이 발생한다.

**동일한 리전 내의 RDS 읽기 복제본의 경우, 해당 비용을 지불하지 않는다**
(가용 영역은 상관이 없다.)
즉, 같은 리전의 다른 AZ에 읽기 복제본을 만들어도 데이터 전송 비용이 발생하지 않아요! 🆓

## 🛡️ RDS 다중 AZ (재해 복구)

RDS 다중 AZ는 재해 복구를 위한 기능이다. 

- **동기식** 복제
- 하나의 DNS 이름 - 대기로 자동 앱 장애 조치
- **가용성** 향상 - 알아서 복제본을 마스터로 승격시켜 준다. 
- AZ 손실, 네트워크 손실, 인스턴스 또는 스토리지 장애 발생 시 장애 조치
- 스케일링에는 사용되지 않음
- 참고: 읽기 복제본을 다중 AZ로 설정하여 재해 복구(DR)에 사용할 수도 있다

## 🔄 RDS - 단일 AZ에서 다중 AZ로 전환

단일 AZ에서 다중 AZ로 전환하는 것은 **제로 다운타임 작업**이다 (DB를 중지할 필요가 없다)

데이터베이스에 대해 "수정"을 클릭하기만 하면 된다.

내부적으로 다음과 같은 일이 일어난다

1. 기본 DB의 스냅샷이 생성된다. 
2. 새 AZ에서 스냅샷으로부터 새 DB가 만들어진다. 
3. 두 데이터베이스 간에 동기화가 설정된다 (만들어진 후에)

## 👑 RDS Custom

RDS는 OS 및 데이터베이스 사용자 지정기능에 접근할 수 없지만(RDS에서 다 해주기 때문에) RDS Custom은 가능하다.

**OS 및 데이터베이스 사용자 지정이 가능한 관리형 Oracle 및 Microsoft SQL Server 데이터베이스**이다

RDS: AWS에서 데이터베이스 설정, 운영 및 확장을 자동화, OS 설정 -> 사용자 지정 그능에 접근할 수 없다.

Custom: 다음과 같은 작업을 하기 위해 기본 데이터베이스 및 OS에 액세스할 수 있다:
- 설정 구성   
- 패치 설치
- 기본 기능 활성화
- **SSH** 또는 **SSM Session Manager**를 사용하여 기본 EC2 인스턴스에 액세스

사용자 지정을 수행하기 위해 **자동화 모드를 비활성화**해야 한다. 그리고 이제 base EC2 인스턴스에 접근이 가능하므로 문제가 쉽게 발생할 수 있기 때문에 먼저 DB 스냅샷을 찍는 것이 좋다

**RDS vs. RDS Custom**:

- RDS: 전체 데이터베이스와 OS가 AWS에 의해 관리됨
- RDS Custom: 기본 OS 및 데이터베이스에 대한 전체 관리자 액세스 권한

## 🚀 Amazon Aurora - 시험에 많이 나옴 

Aurora는 AWS의 독점 기술이다 (오픈 소스가 아님)!

Postgres와 MySQL은 모두 Aurora DB로 지원된다 (즉, 드라이버는 Aurora가 Postgres나 MySQL 데이터베이스인 것처럼 작동하는 것이다.)

Aurora는 "AWS 클라우드 최적화"되어 있으며, RDS의 MySQL보다 5배, RDS의 Postgres보다 3배 이상의 성능 향상이 있다. 

Aurora 스토리지는 10GB 단위로 최대 128TB까지 자동으로 늘어난다. 

Aurora는 최대 15개의 복제본을 가질 수 있으며 복제 프로세스는 MySQL보다 빨라요 (복제 지연 10ms 미만)

Aurora의 장애 조치는 즉각적이에요. 고가용성(HA)이 기본 제공돼요!

Aurora는 RDS보다 비용이 더 많이 들어요 (20% 더 비쌈) - 하지만 더 효율적이에요!

## 🔍 Aurora 고가용성 및 읽기 확장

무언가를 기록할 때마다 
![[스크린샷 2025-05-04 오후 4.30.36.png]]
3개의 AZ에 걸쳐 데이터 6개 복사본:

- 쓰기에는 6개 중 4개 복사본 필요
- 읽기에는 6개 중 3개 복사본 필요
- 백엔드에서 P2P 복제로 자가 복구를 한다. 
- 스토리지는 수백 개의 볼륨에 걸쳐 분산 (striped 형식으로 )

하나의 Aurora 인스턴스가 쓰기를 담당한다 (마스터) 

마스터의 자동 장애 조치는 30초 미만, 블록 단위로 자가 복구 또는 확장을 한다. 

마스터 + 최대 15개의 Aurora 읽기 복제본이 읽기를 처리한다. 

**교차 리전 복제 지원**
 
## 🔄 Aurora 자동 확장

![[스크린샷 2025-05-04 오후 4.33.25.png]]

Aurora는 다음과 같은 엔드포인트를 제공한다:

- **Writer 엔드포인트**: 마스터를 가리킴
	- 장애 조치 후에도 클라이언트는 writer 엔드포인트와 상호작용을 한다. 
- **Reader 엔드포인트**: 연결 로드 밸런싱이 있는 읽기 복제본을 가리킴
	- 클라이언트가 리더 엔드포인트에 연결될 때마다 읽기 전용 복제본 중 하나로 연결된다. (로드벨런싱을 해주는 것이다 - 문장 level이 아닌 connection level) 

읽기 복제본은 그림에서 보이는 것과 같이 Auto Scaling을 한다. 항당 15개를 가지고 있는 것은 아니다. 

## ✨ Aurora의 특징

- 자동 장애 조치
- 백업 및 복구
- 격리 및 보안
- 산업 규정 준수
- 버튼식 확장
- 제로 다운타임 자동 패치
- 고급 모니터링
- 정기 유지 관리
- 백트랙: 백업을 사용하지 않고도 언제든지 데이터 복원 가능

데이터 베이스 만들기
Aurora 
버전 선택, 자격 증명 정보, storage 설정, 인스턴스 설정하기, Replica를 만들어서 Multi AZ 배포를 할 것이냐 , VPC 설정, IP 등을 완료해서 만들기

## 🌟 Aurora 복제본 - 자동 스케일링
![[스크린샷 2025-05-04 오후 5.04.55.png]]

### 🎯 핵심 구성 요소

- **Writer Endpoint**: 데이터 쓰기
- **Reader Endpoint**: 데이터 읽기 (Replicas)
- **공유 스토리지 볼륨**: 모든 데이터가 있는 곳

요청이 많아지면? CPU 사용량을 체크해서 복제본을 자동으로 추가한다 마찬가지로 요청이 줄어들면? 필요없는 복제본은 없어지게 된다

#### 🎨 Aurora 사용자 정의 엔드포인트 - 내 맘대로 설정하기

특정 작업을 위한 전용 엔드포인트를 만들 수 있다

**예시 시나리오:**

- 분석 쿼리는 `db.r5.2xlarge` 인스턴스로만 보내고 싶어요!
- 일반 쿼리는 `db.r3.large` 인스턴스로 보내고 싶어요!

이렇게 하면 무거운 분석 작업이 일반 서비스에 영향을 주지 않는다 👍

> 💡 **Pro Tip**: 사용자 정의 엔드포인트를 만들면 기본 Reader Endpoint는 쓰이지 않는다.

#### ⚡ Aurora Serverless - 서버 걱정 없는 데이터베이스!

자동화된 데이터베이스 instantiation과 실제 사용량에 따른 Auto Scaling을 제공한다. -> 예측 불가능한 업무량에 대응하는 것을 도와준다. 

🎯 장점들:

- **자동 인스턴스화**: 필요할 때 자동으로 켜진다
- **자동 스케일링**: 사용량에 따라 크기가 조절된다
- **용량 계획 불필요**: 미리 계획할 필요가 없다
- **초 단위 과금**: 사용한 만큼만 낸다.

미리 용량을 설정하지 않아도 되는 것이다. 

🎪 적합한 사례:

- 가끔씩만 사용하는 개발/테스트 환경 
- 예측할 수 없는 트래픽 패턴
- 새로운 앱으로 아직 사용량을 모를 때

#### 🌍 Global Aurora 

##### 🔄 두 가지 방식:

**1. Aurora Cross Region Read Replicas**

- 간단하게 설정 가능
- 재해 복구(DR)에 좋다

**2. Aurora Global Database (추천! 👍)**

- 1개의 기본 리전 (읽기/쓰기)
- 최대 5개의 보조 리전 (읽기 전용)
- 복제 지연 시간 < 1초! (정말 빠르다 🚄)
- 각 보조 리전당 최대 16개의 읽기 복제본 <- 전 세계에서 오는 읽기 업무량에 따른 대기 시간을 줄이는 데 도움을 준다. 
- 재해 시 다른 리전 승격: RTO < 1분!

> 💡 **놀라운 사실**: 일반적인 교차 리전 복제(한 리전에서 다른 리전으로의 복제)는 1초도 안 걸린다.

이 표현을 시험에서 본다면 Global Aurora 를 사용하라는 것이다. 

#### 🤖 Aurora Machine Learning - AI가 내 데이터베이스에

이제 Aurora에서 바로 머신러닝을 쓸 수 있다
SQL 쿼리만으로 우리의 응용프로그램에 머신러닝 기반의 예측을 추가할 수 있다는 것이다. 

##### 🔧 지원 서비스:

아래 서비스와 통합 지원된다.
- **Amazon SageMaker**: 모든 ML 모델과 함께 사용
- **Amazon Comprehend**: 감정 분석 전문가

##### 💼 실제 사용 사례:

- 사기 거래 탐지 🕵️‍♀️
- 광고 타겟팅 🎯
- 상품 추천 시스템 🛍️
- 고객 감정 분석 😊😐😠

**작동 방식:**
![[스크린샷 2025-05-04 오후 5.19.10.png]]

1. 앱이 Aurora에 SQL 쿼리 보냄 ("이 고객에게 어떤 상품 추천?")
2. Aurora가 데이터를 ML 서비스로 보냄
3. ML 서비스가 예측 결과 반환
4. Aurora가 결과를 앱에 전달

####  🔄 Babelfish for Aurora PostgreSQL 

강의에서 다루지는 않았다.

마이크로소프트 SQL Server에서 Aurora PostgreSQL로 이사가고 싶을 때 도와주는 서비스이다. 

### 🎯 핵심 기능:

- MS SQL Server 명령어(T-SQL)를 이해 할 수 있다. 
- 코드 변경이 거의 없다. (기존 SQL Server 드라이버 그대로 사용)
- AWS SCT와 DMS로 데이터 마이그레이션 후 바로 사용 가능

> 💡 **이름의 비밀**: Babelfish는 "은하수를 여행하는 히치하이커를 위한 안내서"에 나오는 번역 물고기이다

## 💾 RDS 백업

2가지 방식이 있다.
#### 🔄 자동 백업:

- **일일 전체 백업**: 매일 지정된 백업 시간에 실행
- **트랜잭션 로그 백업**: 5분마다! (꽤 자주한다)
- **Point-in-Time 복원**: 최대 35일 전까지의 어느 시점으로든 복원 가능
- **보존 기간**: 1~35일 (0으로 설정하면 백업 비활성화)

#### 📸 수동 스냅샷:

- 원할 때 언제든지 찍을 수 있다. 
- 원하는 만큼 오래 보관 가능

> 💡 **꿀팁**: RDS가 중지된 상태에서도 스토리지 비용은 나간다. 오래 중지할 거면 스냅샷 찍고 삭제하는 게 경제적이다. 스냅샷은 RDS 의 실제 스토리지 바용보다 훨씬 저렴하기 때문이다. 

### 🎯 Aurora 백업 - 더 강력한 보호!

#### 자동 백업:

- 1~35일 보존 (비활성화 불가능 - 안전이 최우선)
- Point-in-Time 복원 지원 : 해당 기간의 어느 시점으로든 복구할 수 있다. 

#### 수동 스냅샷:

- RDS와 동일하게 원하는 만큼 보관 가능
- 사용자가 수동으로 트리거 

##### 🔄 RDS & Aurora 복원 옵션 - 다양한 복원 방법

📌 중요한 사실:

**백업이나 스냅샷을 복원하면 항상 새로운 데이터베이스가 생성된다**

S3에서 MySQL RDS 복원:

1. 온프레미스 DB 백업 생성
2. S3에 백업 파일 업로드
3. 새 RDS 인스턴스로 복원

S3에서 Aurora MySQL 복원:

1. Percona XtraBackup라는 외부 소프트웨어로 백업을 생성
2. S3에 백업 파일 업로드
3. 새 Aurora 클러스터로 복원

RDS Mysql로 복원할 때는 데이터베이스의 백업만 있으면 되는데 Aurora MySQL에서는 Percona XtraBackup로 백업을 한 다음에 S3에서 Aurora DB 클러스터로 백업을 한다. 

=) Aurora 클러스터 : 여러 데이터베이스 인스턴스들이 하나의 공유 스토리지를 사용하는 그룹이다. Primary Instance (Writer), Aurora Replicas (Readers), Shared Storage Volume으로 구성된다.

#### 🔬 Aurora DB 클로닝 - 똑똑한 복제 기술!

![[스크린샷 2025-05-04 오후 5.34.04.png]]

⚡ 특징:

- 스냅샷 & 복원보다 훨씬 빠르다
- **Copy-on-Write** 프로토콜 사용
- 처음 데이터베이스 복제본을 만들 때는, 원래 데이터 베이스 클러스터와 동일한 데이터 볼륨을 사용한다. (복사 없음!)
- 프로덕션에서 스테이징으로 업데이트가 이루어지면 새로운 추가 스토리지가 할당되고, 데이터가 복사 및 분리가 된다. 데이터가 수정될 때만 분리되는 것이다. 

## 🔒 RDS & Aurora 보안

##### 🔐 저장 데이터 암호화:

데이터 볼륨에 암호화된다는 것이다. 
- AWS KMS로 마스터 & 복제본 암호화
- 데이터베이스를 처음 실행할 때 정의된다. 
- 마스터가 암호화되지 않으면 복제본도 암호화 불가 
- 암호화되지 않은 DB를 암호화하려면? 스냅샷 → 암호화된 데이터베이스 형태로 데이터베이스 스냅샷 복원

##### 🔒 전송 중 암호화:

- TLS 기본 지원
- AWS TLS 루트 인증서 사용
##### 🎫 IAM 인증:

- 사용자명/비밀번호 대신 IAM 역할 사용 가능
##### 🛡️ 보안 그룹:

- 네트워크 접근 제어
##### 🚫 SSH:

- RDS Custom 제외하고는 SSH 접근 불가
##### 📝 감사 로그:

- CloudWatch Logs로 전송하여 장기 보관 가능

## 🔌 Amazon RDS Proxy

프록시를 사용하면 애플리케이션이 데이터베이스 내에서 데이터베이스 연결 풀을 형성하고 공유할 수 있다. 
애플리케이션을 RDS 데이터베이스 인스턴스에 일일이 연결하는 대신 프록시가 하나의 풀에 연결을 모아 RDS 데이터베이스 인스턴스로 가는 연결이 줄어든다.
#### 🌟 주요 기능:

- DB 연결 풀링 & 공유
- CPU/RAM 부담 감소
- 연결 문제 & 타임아웃 최소화
- 서버리스, 자동 스케일링, 고가용성(Multi-AZ)
- **장애 조치 시간 66% 단축!** ⚡
	- 메인 RDS 데이터베이스 인스턴스에 애플리케이션을 모두 연결하고 장애 조치를 각자 처리하게 하는 대신, 장애 조치와 무관한 RDS 프록시에 연결하는 것이다. 
- 대부분의 앱에서 코드 변경 불필요
	- RDS 데이터베이스 인스턴스나 Aurora 데이터베이스에 연결하는 대신 RDS 프록시에 연결하기만 하면 된다. 

#### 🛡️ 보안 특징:

- IAM 인증 강제한다 
- AWS Secrets Manager에 자격 증명을 안전하게 저장
- VPC 내에서만 접근 가능 (public 접근 불가) - 인터넷을 통해 RDS 프록시에 연결할 수 없으니 보안이 좋다. 

#### 📊 지원 데이터베이스:

- RDS: MySQL, PostgreSQL, MariaDB, SQL Server
- Aurora: MySQL, PostgreSQL

> 💡 **사용 사례**: Lambda 함수에서 데이터베이스 접근할 때 특히 유용하다 - 순식간에 람다 함수가 많아지고 줄어드니까

>[!note]
>RDS 데이터베이스 인스턴스의 연결을 최소화 할 수 있고
>장애 조치 시간을 최대 66%까지 감소시킬 수 있다. 
>IAM 인증을 강제 하는데 사용된다. 

# 🚀 ElastiCache 

ElastiCache는 관리형 Redis 또는 Memcached를 제공하는 서비스이다. RDS가 관계형 DB를 관리하듯이 ElastiCache는 캐시를 관리해준다 

### 🎯 주요 이점: 

- 인메모리 데이터베이스로 초고속 성능
- 읽기 집약적 워크로드의 DB 부하 감소
- 애플리케이션을 상태 비저장으로 만들기
- AWS가 모든 관리 작업 처리 (패치, 모니터링, 백업 등)

> ⚠️ **주의**: ElastiCache 사용하려면 애플리케이션 코드 수정이 필요하다



#### 🏗️ ElastiCache 아키텍처 패턴

##### 1️⃣ DB 캐시 패턴 (Lazy Loading):

![[스크린샷 2025-05-04 오후 5.55.18.png]]

```
1. 앱이 ElastiCache에서 데이터 조회
2. 캐시 미스? → RDS에서 데이터 가져오기
3. 가져온 데이터를 ElastiCache에 저장
4. 다음 요청에서는 캐시에서 바로 제공!
```

##### 2️⃣ 세션 스토어 패턴:
![[스크린샷 2025-05-04 오후 5.56.39.png]]
```
1. 사용자가 앱에 로그인
2. 세션 정보를 ElastiCache에 저장
3. 다른 인스턴스에서도 같은 세션 사용 가능
4. 완벽한 상태 비저장 아키텍처!
```

#### ⚔️ Redis vs Memcached

##### 🔴 Redis:

- Multi-AZ with Auto-Failover(자동 장애 조치) ✅
- 읽기 복제본으로 확장성 & 고가용성 ✅
- AOF 지속성으로 데이터 내구성 제공 ✅
- 백업 & 복원 기능 ✅
- Sets, Sorted Sets 지원 -> 리더보드를 생성하는 데 매우 유용하다 ✅

##### 🟡 Memcached:

- 멀티 노드로 데이터 샤딩(데이터 분할) ✅
- 고가용성 없음 ❌
- 지속성 없음 ❌
- 백업 & 복원 기능은(서버리스만있고 ElasticCache에서의 자체 관리버전에는 없다 ) ⚠️
- 멀티스레드 아키텍처 ✅

> 💡 **선택 기준**: 데이터 지속성과 고가용성이 필요하면 Redis, 단순 캐싱만 필요하면 Memcached!

## 🔒 ElastiCache 보안

### Redis 보안:

- IAM 인증 지원 (Redis 6+)
- Redis AUTH (비밀번호/토큰)
- SSL/TLS 전송 중 암호화
- 보안 그룹으로 네트워크 접근 제어

EC2 인스턴스와 클라이언트가 있는 경우 Redis Auth를 사용하여 Redis 클러스터에 연결할 수 있다. 

### Memcached 보안:

- SASL 기반 인증
- 보안 그룹으로 네트워크 접근 제어

## 🎯 ElastiCache 사용 패턴

### 캐싱 전략:

1. **Lazy Loading**: 캐시 미스 시에만 데이터를 캐시에 로드 (그래서 게으르다는 것이다)
2. **Write Through**: 쓰기 작업 시 캐시도 업데이트
3. **Session Store**: TTL로 임시 세션 데이터 관리

> 💭 **명언**: "컴퓨터 과학에서 어려운 두 가지: 캐시 무효화와 이름 짓기"

##### 🎮 Redis 실전 사용 사례 - 게임 리더보드

게임 순위표를 만들고 싶을떄 Redis Sorted Sets를 이용하면 된다 
![[스크린샷 2025-05-04 오후 6.10.00.png]]
요소가 추가될때마다, 실시간으로 순위를 매긴 다음 올바른 순서로 추가된다. 

특징:

- 고유성 보장
- 자동 정렬
- 실시간 순위 업데이트
- 초고속 성능

여러 ElastiCache 클러스터로 분산하면 대규모 게임도 문제없이 가능하다.

## 친숙한 포트 목록 (강의자료)
아래는 한 번 이상은 확인하는 게 좋은 **표준** 포트의 목록입니다. 외울 필요는 없습니다(시험에 나오지 않을 거예요). 하지만 **중요한 포트(HTTPS - 443번 포트)와 데이터베이스 포트(PostgreSQL - 5432번 포트)를 구분할 수는 있어야 합니다.**

**중요한 포트:**

- FTP: 21
    
- SSH: 22
    
- SFTP: 22 (SSH와 같음)
    
- HTTP: 80
    
- HTTPS: 443
    

**vs RDS 데이터베이스 포트:**

- PostgreSQL: 5432
    
- MySQL: 3306
    
- Oracle RDS: 1521
    
- MSSQL Server: 1433
    
- MariaDB: 3306 (MySQL과 같음)
    
- Aurora: 5432 (PostgreSQL와 호환될 경우) 또는 3306 (MySQL과 호환될 경우)
  

> 스트레스받으면서 외우지 마세요. 표준 포트 목록은 오늘 한 번 확인하고 시험 전에 한 번 더 읽어보기만 해도 시험에서는 문제없을 거예요. :)
> 
> “중요한 포트”와 “RDS 데이터베이스 포트”를 구분할 줄만 알면 됩니다.