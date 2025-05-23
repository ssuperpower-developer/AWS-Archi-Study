## 확장성 (Scalability)

조정을 통해 시스템이 더 많은 양을 처리할 수 있는 능력

### 수직 확장 (Vertical Scaling)
단일 인스턴스의 퍼포먼스를 올리는 것. 예시로 t2.micro → t2.large
- 주로 분산이 어려운 DB 등에 사용
- 하드웨어 업그레이드에 한계가 존재

### 수평 확장 (Horizontal Scaling)
인스턴스 수를 늘리는 것.
- 작업 분산 처리 가능
- 로드 밸런서 필요
- 클라우드 환경에서 용이        

## 고가용성 (High Availability)

시스템의 일부에 장애가 발생해도 계속 서비스를 제공할 수 있는 능력 
- **여러 Availability Zone에 인스턴스 분산
- RDS 다중 AZ, ELB + Auto Scaling Group

여기서도 스케일링과 동일하게 Vertical Scaling과 Horizontal Scaling을 사용한다.
- Vertical Scaling으로 인스턴스의 퍼포먼스를 늘리거나 줄이면?
  -> Scale up / Down
- Horizontal Scaling으로 인스턴스의 수를 늘리거나 줄이면?
  -> Scale Out / In
