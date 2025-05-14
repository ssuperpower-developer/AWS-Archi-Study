
### 📦 객체 스토리지

|서비스|특징|사용 예시|
|---|---|---|
|**Amazon S3**|객체 단위 저장, API 기반, 스토리지 클래스 다양|백업, 정적 웹 호스팅|
|**Amazon S3 Glacier**|장기 아카이브용, 느린 복구|아카이빙, 감사 로그 저장|

---

### 💽 블록 스토리지

|서비스|특징|사용 예시|
|---|---|---|
|**EBS (Elastic Block Store)**|EC2 전용 볼륨, 한 인스턴스에 마운트|DB 저장소, OS 볼륨|
|**EBS Multi-Attach**|IO1/IO2에서만 가능, 여러 인스턴스에 연결|클러스터 구성|
|**EC2 Instance Store**|임시 로컬 디스크, 휘발성|캐시, 고속 임시 저장소|

---

### 📁 파일 스토리지

|서비스|특징|사용 예시|
|---|---|---|
|**Amazon EFS**|POSIX 호환, 다중 AZ 지원|공유 파일 시스템|
|**FSx for Windows File Server**|SMB/NTFS, AD 통합|Windows 파일 서버|
|**FSx for Lustre**|병렬 처리, HPC/ML 용도|대규모 분석 워크로드|
|**FSx for NetApp ONTAP**|NFS/SMB/iSCSI, 고급 기능 포함|NAS 이전, 압축/중복 제거|
|**FSx for OpenZFS**|OpenZFS 기반, 고성능|ZFS 기반 환경 마이그레이션|

---

## 🌐 하이브리드 스토리지 연계

|서비스|특징|사용 예시|
|---|---|---|
|**AWS Storage Gateway**|온프레미스 ↔ 클라우드 가교 역할|파일/볼륨/테이프 게이트웨이|
|- S3/FSx File Gateway|로컬 파일을 S3/FSx와 동기화|공유 디렉터리 캐시|
|- Volume Gateway|iSCSI → S3 + EBS 스냅샷|블록 백업|
|- Tape Gateway|테이프 백업 → VTL + Glacier|기존 백업 시스템 유지|

---

## 🔄 데이터 이동 서비스

|서비스|특징|사용 예시|
|---|---|---|
|**AWS Transfer Family**|FTP, FTPS, SFTP 제공|외부 시스템과 파일 전송|
|**AWS DataSync**|스케줄 기반 대용량 동기화|S3 ↔ FSx, 온프레미스 ↔ AWS|
|**AWS Snow Family**|물리 장비로 오프라인 전송|초대용량 데이터 이전|
|- Snowcone|소형 장치, DataSync 에이전트 내장|엣지 연산 + 전송|
|- Snowball|수십~수백 TB|대용량 마이그레이션|
|- Snowmobile|엑사바이트 규모|초대형 데이터센터 이전|

---

## 📌 시험 대비 핵심 포인트 요약

- **S3** = 객체 스토리지 / **Glacier** = 아카이브
- **EBS** = EC2 1:1, **Instance Store** = 휘발성 로컬
- **EFS** = POSIX, 다중 AZ 공유
- **FSx 시리즈** = 특정 파일 시스템 필요 시
- **Storage Gateway** = 온프레미스 ↔ AWS 연계
- **Transfer Family** = FTP/SFTP로 S3 접근
- **DataSync** = 스케줄 기반 동기화
- **Snow Family** = 네트워크 불가 상황에서 대량 전송