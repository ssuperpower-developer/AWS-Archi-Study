
## 로드 밸런서란?
![[Pasted image 20250503235709.png]]
- 하나의 엔드포인트로 클라이언트 요청을 받아서 **백엔드 서버들(EC2 등)로 분산 처리**하는 역할을 함.

### 기능과 장점

1. **트래픽 분산**
   각 유저의 요청을 다른 인스턴스로 자동 분배
2. **Health Check**
   특정 인스턴스가 응답하지 않으면 해당 인스턴스로는 요청 전송 ❌
   예: HTTP 4567 포트의 `/health`에서 상태 체크
3. **SSL 종료 (SSL Termination)**
   HTTPS 트래픽을 로드 밸런서에서 처리하여 EC2는 HTTP로 통신
4. **고정 세션 (Sticky Session)**
   쿠키 기반으로 클라이언트 요청을 동일 인스턴스로 유지 가능
5. **보안 강화**
   로드 밸런서는 `0.0.0.0/0`에서 80, 443 포트 허용하고, EC2는 **로드 밸런서 보안 그룹**만 허용
   -> EC2는 오직 로드밸런서에서 오는 요청만 받게됨
        
### ELB의 종류

| 이름                    | 프로토콜                   | 특징             |
| --------------------- | ---------------------- | -------------- |
| **CLB** (Classic)     | HTTP, HTTPS, TCP       | 지금은 Deprecated |
| **ALB** (Application) | HTTP, HTTPS, WebSocket | L7 로드 밸런싱      |
| **NLB** (Network)     | TCP, UDP, TLS          | L4 고성능 트래픽     |
| **GWLB** (Gateway)    | L3 (IP 기반)             | 보안 어플라이언스 통합용  |

---

# ALB

7계층인 Application Layer에서 작동하는 로드밸런서이다. 하나의 ALB로 **여러 애플리케이션 트래픽**을 처리 가능하다는 특징이 있어서 MSA와 컨테이너 기반 애플리케이션에 아주 적합하다!

### 주요 기능

![[Pasted image 20250503235913.png]]

1. **경로 기반 라우팅 (Path-based Routing)**  
   `/users` → 대상 그룹 A, `/search` → 대상 그룹 B
2. **호스트 기반 라우팅 (Host-based Routing)**  
   `one.example.com` → 그룹 A, `other.example.com` → 그룹 B
3. **쿼리 스트링 / 헤더 기반 라우팅**  
    `?platform=Mobile` → 모바일 전용 그룹

### Target Group (대상 그룹)

ALB가 요청을 분배하는 단위를 말한다. 아래 요소들이 타겟 그룹이 될 수 있다. 또한, **Health Check는 타켓 그룹 레벨에서 수행된다.**
- EC2 인스턴스
- ECS 작업(Task)
- Lambda 함수
- 사설 IP 기반 온프레미스 서버
        
### 클라이언트, ALB, EC2의 통신

- ALB에는 **DNS Name**이 부여됨. 예를 들어,`abc.amazonaws.com`
- 클라이언트의 실제 IP는 직접 전달되지 않음  
![[Pasted image 20250504000004.png]]
1. 클라이언트가 ALB의 Public IP로 요청을 보냄. (DNS Name을 통해 이루어짐.)
2. ALB와 EC2는 각자의 Private IP로 통신하며 클라이언트 요청을 수행함.
   따라서, EC2는 클라이언트의 실제 IP를 알 수 없음.
3. `X-Forwarded-For`, `X-Forwarded-Port`, `X-Forwarded-Proto` 헤더로 클라이언트의 정보를 확인 가능.


---
# 네트워크 로드 밸런서 (NLB)

- **4계층**인 TCP, UDP 기반으로 동작하는 **엄청! 고성능인 로드밸런서**이다. 초당 **수백만 건의 요청 처리**가 가능하며, 지연 시간도 짧다. 중요한 특징으로는, 하나의 Availability Zone마다 하나의 고정 IP를 할당받는다. (물론 Elastic IP를 가지고 있다면 사용할 수 있다!)

## 시험에 어떻게 나오는가?

1. 애플리케이션이 하나, 두 개 또는 세 개의 다른 IP로만 접근할 수 있다고 할 때
2. 극한의 성능, TCP 또는 UDP, 고정 IP가 나올 때

## ALB와 비교

작동 방식이 ALB와 아주 유사하다. 보안 그룹과 Target Group을 사용하는 것과 Health Check가 이루어지는 것도 모두 같다. 그래서 ALB를 맨 처음에 배우나보다.. 아래는 NLB만의 차이점이다.

1. **TCP, UDP 지원**
   어플리케이션은 여전히 HTTP로 작동하지만, 클라이언트는 TCP로 요청을 보낼 수 있다.
2. **고정 IP 지원**
   ALB는 로드밸런서에 고정 IP가 생기는 것이 아니라 DNS Name이 할당된다. 따라서 DNS Name에 할당 된 IP가 변할 수 있으므로 고정 IP가 아니다. 반면, NLB에는 고정 IP가 할당된다.
3. **대상 그룹**
   ALB에서 설명한 대상 그룹 모두 동일하게 쓸 수 있다. 단, NLB에서는 ALB를 대상 그룹으로 사용할 수 있다. 이때는 NLB가 ALB 앞에 있는 구조가 된다. 

1, 2, 3번을 조합하면 아래 그림과 같은 구조이다.
NLB는 고정 IP를 제공하고, 빠른 속도로 부하를 처리하는 역할을 수행하고, 뒷 단에 있는 ALB는 HTTP 트래픽을 세부 라우팅 하는 역할을 수행한다.
   ![[Pasted image 20250504004326.png]]

---

# GWLB

**3계층인 IP** 기반으로 작동하는 로드밸런서이다. 다른 것들과는 조금 성격이 다르다. 모든 트래픽이 GWLB를 통과하게 해서 서드파티 보안 어플라이언스를 (방화벽 등) 거치게 하는 것이 주된 목적이다.

## 동작 방식

아래 그림을 이해하는 것이 가장 중요하다.
1. 유저가 보내는 **모든 트래픽이** GWLB를 통과한다.
2. GWLB를 통해 Target Group인 서드파티 보안 어플라이언스로 트래픽이 전송된다.
3. 보안에 위배된 트래픽은 버려지고, 적합한 트래픽만 다시 GWLB로 돌려보내진다.
4. GWLB는 돌아온 트래픽을 ALB나 어플리케이션에 전달한다.
![[Pasted image 20250504205406.png]]

## 시험에 나오는 내용

- Target Group은 보안을 담당하는 서드파티 어플라이언스이다.
  이런 어플리케이션이 돌아가는 EC2 ID나 Private IP를 등록하면 된다.
- 포트는 6081번, 프로토콜은 **GENEVE**
- EC2나 ALB 앞 단에 보안 계층 추가가 필요하면 GWLB 고려


