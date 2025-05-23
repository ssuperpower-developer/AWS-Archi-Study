
## 1. 첫 번째 단계: 단일 EC2 인스턴스 (POC 레벨) 🌱

처음에는 간단하게 시작해보자:

- T2 마이크로 인스턴스 하나로 시작 (가장 작은 사이즈!)
- 탄력적 IP 연결 (재시작해도 IP 주소가 바뀌지 않도록)
- 사용자가 "지금 몇 시야?"라고 물으면 서버가 시간을 알려줌

이 구성은 초기에는 잘 작동했지만, 사용자가 늘어나면서 금방 한계에 도달하게 된다고 해보자 T2 마이크로는 CPU 크레딧 방식으로 작동하기 때문에 트래픽이 갑자기 증가하면 성능이 급격히 떨어진다.

## 2. 수직 확장: 더 큰 인스턴스로 업그레이드 📈
![[스크린샷 2025-05-07 오후 10.49.00.png]]
- T2 마이크로 → M5 large로 인스턴스 타입 변경
- 더 많은 CPU와 메모리 확보!
- 탄력적 IP 덕분에 동일한 IP 주소 유지

**주의점**: 이 과정에서 인스턴스를 중지하고 다시 시작해야 했기 때문에 다운타임이 발생했다. 사용자들이 잠시 서비스에 접근할 수 없었던 것이다. 이건 프로덕션 환경에서는 좋지 않은 상황! 😫

## 3. 수평 확장: 여러 인스턴스 추가 ↔️

![[스크린샷 2025-05-07 오후 10.49.20.png]]
수직적이 아닌 수평적으로 확장해보자
- 여러 M5 large 인스턴스 추가 (3개의 인스턴스)
- 각 인스턴스마다 탄력적 IP 할당 (총 3개의 IP)

**문제점**:

- 사용자들이 세 가지 다른 IP 주소를 알아야 함 (사용자 경험 최악! 🙅‍♀️)
- AWS에서는 계정당 지역별로 탄력적 IP를 5개까지만 기본 제공 (확장성 한계)
- 관리해야 할 인프라가 증가 (운영 복잡성 증가)

## 4. DNS 활용: Route 53 도입 🌐

위의 문제를 해결하기 위해 DNS를 활용해보자

![[스크린샷 2025-05-07 오후 10.49.47.png]]

- WhatIsTheTime.com 도메인 설정
- A 레코드로 3개의 EC2 인스턴스 IP 연결 (TTL 1시간)
- 사용자들은 이제 도메인 이름만 기억하면 됨

**새로운 문제점**:

- TTL이 1시간이라, 인스턴스를 제거하거나 추가할 때 최대 1시간 동안 사용자들에게 장애가 발생할 수 있음
- 일부 사용자는 이미 다운된 인스턴스에 계속 접속 시도 (DNS 캐싱 때문)
- 인스턴스 자동 관리 기능 부재

## 5. 로드 밸런싱 도입: ELB 추가 ⚖️

![[스크린샷 2025-05-07 오후 10.51.10.png]]

- Elastic Load Balancer(ELB) 추가
- EC2 인스턴스를 퍼블릭 → 프라이빗으로 변경
- Route 53에서 A 레코드 대신 별칭(Alias) 레코드 사용
- 로드 밸런서에 헬스 체크 기능 추가

**개선된 점**:

- 사용자는 항상 도메인에 접속하고, 로드 밸런서가 트래픽을 분산
- 비정상 인스턴스는 자동으로 트래픽 전달에서 제외됨
- 보안 그룹을 활용해 EC2 인스턴스는 로드 밸런서의 트래픽만 허용

## 6. 자동 확장: Auto Scaling Group 도입 🤖

![[스크린샷 2025-05-07 오후 10.51.24.png]]

- Auto Scaling Group(ASG) 설정
- 트래픽에 따라 인스턴스 수 자동 조절
- 피크 시간에 자동으로 확장, 한가한 시간에 자동으로 축소

**중요 이점**:

- 수요에 따른 자동 확장/축소로 비용 최적화
- 수동 관리 필요성 감소
- 장애 인스턴스 자동 교체

**여기서 잠깐!** 💡 Auto Scaling의 경우, CloudWatch 지표를 활용해서 CPU 사용률, 네트워크 트래픽, ELB 요청 수 등 다양한 조건에 따라 스케일링 정책을 설정할 수 있다. 예를 들어 "CPU 사용률이 70%를 넘으면 인스턴스 추가, 30% 미만이면 인스턴스 감소" 같은 규칙을 만들 수 있다

## 7. 고가용성: 멀티 AZ 구성 🏢🏢🏢

마지막으로 해줘야 하는 작업이 장애 대응력 강화이다.
![[스크린샷 2025-05-07 오후 10.53.19.png]]
- 여러 가용 영역(AZ)에 인스턴스 분산 배치
- ELB도 멀티 AZ 구성
- 각 AZ마다 최소 1개 이상의 인스턴스 유지

**결정적 이점**:

- 한 가용 영역이 완전히 다운되어도 서비스 계속 운영 가능
- 지진이나 정전 같은 물리적 재해에도 대응 가능
- AWS의 SLA(서비스 수준 계약) 향상

## 8. 비용 최적화: 예약 인스턴스 & 스팟 인스턴스 활용 💰

진짜 찐찐막으로 비용을 최적화 해보자

- 기본 용량(최소 2개 인스턴스)은 예약 인스턴스로 구매
- 추가 용량은 온디맨드 또는 스팟 인스턴스로 처리
- 예측 가능한 워크로드는 예약, 변동성 있는 워크로드는 스팟 활용

**메모** 📝 예약 인스턴스를 사용하면 온디맨드 대비 최대 72%까지 비용을 절감할 수 있다. 스팟 인스턴스는 90%까지 절감 가능하지만 언제든 종료될 수 있다는 위험이 있다는 것을 기억하자. Auto Scaling Group에서는 이런 다양한 구매 옵션을 혼합해서 사용할 수 있다는 점도 기억하자

## 요약: 아키텍처 진화의 교훈 🎓


1. **점진적 확장**: 작게 시작해서 필요에 따라 확장하는 전략 (T2 마이크로 → 복잡한 멀티 AZ)
2. **가용성 향상**: 단일 인스턴스 → 로드 밸런싱 → 멀티 AZ로 진화하며 가용성 극대화
3. **자동화의 중요성**: 수동 관리에서 Auto Scaling으로 전환하여 운영 부담 감소
4. **비용 최적화**: 예약 인스턴스와 스팟 인스턴스를 활용한 지능적 비용 관리
5. **아키텍처 선택의 트레이드오프**: 각 단계마다 비용, 복잡성, 관리 용이성 사이의 균형

## 추가 고려사항: 실전에서 더 생각해볼 점 🧩

실제 프로덕션 환경에서는 다음 사항도 고려해볼 수 있다:

- **CDN 도입**: CloudFront를 추가하여 글로벌 사용자의 지연 시간 단축
- **서버리스 옵션**: Lambda + API Gateway로 구현하여 EC2 인스턴스 없이 확장 가능
- **컨테이너화**: ECS 또는 EKS를 사용하여 Docker 컨테이너로 배포
- **모니터링 강화**: CloudWatch 대시보드, 알람 설정으로 시스템 상태 실시간 파악
- **재해 복구**: 다중 리전 아키텍처로 리전 장애에도 대응 가능

솔루션 아키텍트로서 여러분의 역할은 비즈니스 요구사항을 이해하고, 적절한 기술 솔루션을 선택하며, 확장성, 가용성, 비용 간의 균형을 맞추는 것이다 ✨

