
## 1. S3 스토리지 클래스 전환 개요 🔄

Amazon S3에서는 객체를 여러 스토리지 클래스 간에 자유롭게 전환할 수 있다:

```
Standard → Standard-IA → Intelligent-Tiering → One Zone-IA → Glacier Flexible Retrieval → Glacier Deep Archive
```

이러한 전환은:

- 수동으로 직접 수행하거나
- 수명 주기 규칙을 통해 자동화할 수 있다.

**핵심 포인트!** 💡 전환은 일반적으로 자주 액세스하는 스토리지에서 덜 자주 액세스하는 저렴한 스토리지로 이루어진다. 이는 데이터의 가치와 접근성 요구 사항이 시간이 지남에 따라 변하는 것을 반영한다.

## 2. S3 수명 주기 규칙 구성하기 📝

수명 주기 규칙은 객체의 자동 전환 및 만료를 관리하는 강력한 도구이다:

### 2.1 수명 주기 작업 유형

1. **전환 작업(Transition Actions)**:
    - 객체를 한 스토리지 클래스에서 다른 클래스로 이동
    - 예: "생성 후 30일 경과한 객체를 Standard-IA로 이동"
    - 예: "생성 후 90일 경과한 객체를 Glacier로 이동"
2. **만료 작업(Expiration Actions)**:
    - 특정 시간이 지난 후 객체 삭제
    - 예: "생성 후 365일 경과한 액세스 로그 파일 삭제"
    - 이전 버전의 객체 삭제 (버전 관리 활성화 시)
    - 불완전한 멀티파트 업로드 삭제

### 2.2 수명 주기 규칙 필터링

규칙은 다음과 같은 조건으로 필터링하여 적용할 수 있습니다:

- **접두사(Prefix)**: 특정 폴더나 경로에 있는 객체에만 적용
    - 예: `images/` 접두사를 가진 모든 객체
- **객체 태그(Object Tags)**: 특정 태그가 있는 객체에만 적용
    - 예: `department=finance` 태그가 있는 객체
- **객체 크기(Object Size)**: 특정 크기 범위에 있는 객체에만 적용 (최신 기능)
    - 예: 128KB 보다 큰 객체에만 적용
- **전체 버킷**: 버킷 내 모든 객체에 적용

**실무 팁!** 🌟 접두사와 태그를 함께 사용하면 매우 세밀한 수명 주기 관리가 가능하다. 예를 들어, `logs/` 접두사와 `critical=true` 태그를 가진 객체에 대해서만 특별한 보존 정책을 적용할 수 있다.

## 3. 실제 사용 시나리오와 해결책 🧩

### 3.1 프로필 이미지 및 섬네일 관리

**시나리오**:

- EC2 애플리케이션에서 사용자 프로필 사진 업로드 후 섬네일 생성
- 섬네일은 쉽게 재생성 가능하며 60일 동안만 필요
- 원본 이미지는 60일 동안 즉시 접근 가능해야 하고, 그 후에는 최대 6시간 지연 허용 가능

**최적의 해결책**:

1. **원본 이미지**:
    - 스토리지 클래스: S3 Standard
    - 수명 주기 규칙: 60일 후 Glacier Flexible Retrieval로 이전
    - 접두사: `originals/`
2. **섬네일 이미지**:
    - 스토리지 클래스: S3 One Zone-IA (비용 효율적, 재생성 가능) - 접근 속도는 나와 있지 않음
    - 수명 주기 규칙: 60일 후 만료(삭제)
    - 접두사: `thumbnails/`

### 3.2 버전 관리된 삭제 복구 전략

**시나리오**:

- 회사 규정: 삭제된 S3 객체를 30일 동안은 즉시 복구 가능해야 함
- 30일 이후에는 최대 365일까지 48시간 이내 복구 가능해야 함

**최적의 해결책**:

1. **S3 버전 관리 활성화**:
    - 삭제된 객체는 실제로 "삭제 마커"로 표시됨
    - 이전 버전은 보존됨
2. **수명 주기 규칙**:
    - 현재 버전이 아닌 객체를 30일 후 Standard-IA로 이전
    - 현재 버전이 아닌 객체를 90일 후 Glacier Deep Archive로 이전
    - 현재 버전이 아닌 객체를 365일 후 만료(삭제)

**설명!** 📝 이 설정에서는 객체가 삭제되면 "삭제 마커"만 생성되고 실제 데이터는 이전 버전으로 보존된다. 30일 동안은 최신 버전 이전의 버전이 Standard에 남아 있어 즉시 복구 가능하고, 그 후에는 Glacier Deep Archive로 이동하여 48시간 이내 복구가 가능하다.

## 4. S3 Analytics: 최적의 전환 시점 찾기 📊

### 4.1 S3 Analytics의 주요 기능

 - 액세스 패턴 분석을 통해 Standard에서 Standard-IA로의 최적 전환 시점 추천( 엑세스가 줄어들었을 때)
- 일일 업데이트되는 .csv 보고서 생성
- 객체 그룹별 액세스 빈도 시각화
- 비용 절감 기회 식별

### 4.2 S3 Analytics 제한 사항

- Standard와 Standard-IA 클래스 간 전환만 분석 (One Zone-IA, Glacier 등은 분석 안 함)
- 초기 분석 결과가 나오기까지 24-48시간 소요
- 전체 버킷 또는 접두사별로만 필터링 가능 (태그 기반 필터링 불가)

**실무 팁!** 💼 S3 Analytics를 먼저 실행하여 액세스 패턴을 이해한 다음, 이를 바탕으로 수명 주기 규칙을 생성하는 것이 좋다. 일반적으로 Standard-IA로의 전환은 30-60일 사이에 최적인 경우가 많지만, 실제 워크로드는 다를 수 있다.

## 5. 스토리지 클래스 전환 시 고려 사항 ⚠️

### 5.1 전환 비용

스토리지 클래스 간 전환할 때 다음 비용이 발생할 수 있습니다:

- 검색 비용 (객체 읽기)
- 초기 데이터 쓰기 비용
- 최소 보관 기간이 지나지 않은 경우 잔여 기간에 대한 요금

### 5.2 최소 저장 기간

- Standard-IA, One Zone-IA: 30일
- Glacier Instant Retrieval: 90일
- Glacier Flexible Retrieval: 90일
- Glacier Deep Archive: 180일

### 5.3 객체 크기 고려

- 작은 객체(128KB 미만)는 Standard-IA나 One Zone-IA에서 비효율적
- 작은 객체 많은 워크로드는 S3 Intelligent-Tiering 또는 Standard를 고려해야 함
- 최소 스토리지 청구 크기:
    - Standard-IA, One Zone-IA: 128KB
    - Glacier Instant Retrieval: 128KB
    - Glacier Flexible Retrieval: 40KB

## 6. 수명 주기 규칙 설계 모범 사례 👍

### 6.1 데이터 특성별 최적 규칙

**로그 파일**:

- 최근 30일: Standard (빠른 액세스)
- 30-90일: Standard-IA (덜 빈번한 액세스)
- 90-180일: Glacier Flexible Retrieval (아카이브)
- 180일 이후: 만료(삭제) 또는 Deep Archive

**백업 데이터**:

- 최근 30일: Standard
- 30-90일: Standard-IA
- 90-365일: Glacier Flexible Retrieval
- 365일 이후: Glacier Deep Archive

**미디어 자산**:

- 인기 콘텐츠: Standard
- 6개월 이상된 콘텐츠: Standard-IA
- 1년 이상된 콘텐츠: Glacier Instant Retrieval
- 아카이브 콘텐츠: Glacier Flexible Retrieval

### 6.2 규칙 테스트 및 모니터링

1. 소규모 테스트 버킷에서 먼저 규칙 검증
2. 비용 절감과 성능 영향 모니터링
3. S3 스토리지 렌즈로 비용 및 사용량 추세 추적
4. 필요에 따라 규칙 조정 및 최적화

**고급 팁!** 🧠 여러 규칙을 조합하여 복잡한 수명 주기 전략을 구현할 수 있다. 예를 들어, 태그별로 다른 보존 정책을 적용하거나, 중요도에 따라 다른 전환 경로를 설정할 수 있다.

## 7. 복잡한 수명 주기 전략 예시 🧩

### 7.1 데이터 중요도 기반 멀티 티어 전략

**1등급 데이터 (중요한 비즈니스 데이터)**:

- 접두사: `/critical/`
- 태그: `importance=high`
- 전략:
    - 0-90일: Standard (3 AZ)
    - 90-365일: Standard-IA (3 AZ)
    - 365일 이후: Glacier Flexible Retrieval (3 AZ)
    - 만료 없음

**2등급 데이터 (일반 비즈니스 데이터)**:

- 접두사: `/standard/`
- 태그: `importance=medium`
- 전략:
    - 0-30일: Standard
    - 30-180일: Standard-IA
    - 180-730일: Glacier Flexible Retrieval
    - 730일 이후: Glacier Deep Archive

**3등급 데이터 (임시 또는 쉽게 ==재생성== 가능한 데이터)**:

- 접두사: `/temporary/`
- 태그: `importance=low`
- 전략:
    - 0-30일: One Zone-IA
    - 30-90일: Glacier Instant Retrieval
    - 90일 이후: 만료(삭제)

### 7.2 버전 관리된 객체를 위한 복합 규칙

- **현재 버전**:
    - 0-30일: Standard
    - 30-90일: Standard-IA
    - 90일 이후: Glacier Flexible Retrieval
- **이전 버전**:
    - 0-7일: Standard
    - 7-30일: Standard-IA
    - 30-90일: Glacier Flexible Retrieval
    - 90일 이후: 만료(삭제)
- **삭제 마커**:
    - 30일 후 만료(삭제 마커 제거)

## 8. 요약: 핵심 포인트 📌

1. **수명 주기 관리 자동화**:
    - 전환 작업으로 스토리지 클래스 간 객체 이동
    - 만료 작업으로 오래된 객체 자동 삭제
2. **비용 절감 전략**:
    - 접근 빈도가 낮아지는 데이터를 저렴한 스토리지로 이동
    - S3 Analytics를 활용한 최적 전환 시점 결정
3. **유연한 필터링**:
    - 접두사(폴더), 태그, 객체 크기로 세밀한 규칙 적용
4. **실제 사용 사례**:
    - 프로필 이미지와 섬네일 관리
    - 버전 관리된 객체의 복구 전략
5. **주의 사항**:
    - 최소 저장 기간 고려
    - 작은 객체 처리 전략
    - 전환 비용 이해

## S3 요청자 지불

1. 요청자가 익명이어서는 안되고 
2. 요청자가 AWS의 인증을 받아야 한다. 


# Amazon S3 이벤트 알림: 스토리지 이벤트에 실시간 대응하기 🔔

## 1. S3 이벤트 알림이란? 🔍

S3 이벤트 알림은 S3 버킷에서 특정 이벤트가 발생할 때 자동으로 알림을 생성하고 지정된 대상으로 전송하는 기능입니다. 이를 통해 S3와 다른 AWS 서비스 간의 자동화된 워크플로우를 구축할 수 있습니다.

### 1.1 주요 이벤트 유형

S3에서 감지하고 알림을 보낼 수 있는 주요 이벤트 유형:

- **객체 생성 이벤트**:
    - `s3:ObjectCreated:Put`
    - `s3:ObjectCreated:Post`
    - `s3:ObjectCreated:Copy`
    - `s3:ObjectCreated:CompleteMultipartUpload`
- **객체 삭제 이벤트**:
    - `s3:ObjectRemoved:Delete`
    - `s3:ObjectRemoved:DeleteMarkerCreated`
- **객체 복원 이벤트**:
    - `s3:ObjectRestore:Post`
    - `s3:ObjectRestore:Completed`
- **객체 복제 이벤트**:
    - `s3:Replication:OperationFailedReplication`
    - `s3:Replication:OperationCompletedReplication`
- **수명 주기 관련 이벤트**:
    - `s3:LifecycleTransition`
    - `s3:LifecycleExpiration`

### 1.2 이벤트 필터링

이벤트 알림은 다양한 기준으로 필터링할 수 있습니다:

- **접두사(prefix)**: 특정 폴더 경로에 있는 객체만 대상으로 함
    - 예: `images/` 폴더 내 객체만 이벤트 생성
- **접미사(suffix)**: 특정 파일 확장자를 가진 객체만 대상으로 함
    - 예: `.jpg` 확장자를 가진 이미지 파일만 이벤트 생성

**실무 팁!** 💡 접두사와 접미사를 함께 사용하면 더 세밀한 필터링이 가능합니다. 예를 들어, `uploads/images/` 폴더에 있는 `.png` 파일에 대해서만 이벤트를 생성할 수 있습니다.

## 2. 이벤트 알림 대상 옵션 🎯

S3 이벤트 알림을 전송할 수 있는 대상은 총 4가지입니다:

### 2.1 Amazon SNS(Simple Notification Service)

- **용도**: 이메일, SMS, 모바일 푸시 알림 등 다양한 구독자에게 알림 전송
- **특징**: 팬아웃 패턴(하나의 메시지를 여러 수신자에게 전송)에 적합
- **사용 사례**: 중요 파일 업로드 시 관리자에게 이메일 알림, 외부 시스템에 웹훅 전송

### 2.2 Amazon SQS(Simple Queue Service)

- **용도**: 메시지 대기열을 통한 서비스 간 통신
- **특징**: 메시지 지속성, 처리 보장
- **사용 사례**: 업로드된 파일을 **비동기적으로 처리**해야 할 때, 부하 분산

### 2.3 AWS Lambda

- **용도**: 서버리스 컴퓨팅을 통한 즉각적인 파일 처리
- **특징**: 코드 실행, 완전 관리형 컴퓨팅
- **사용 사례**: 이미지 섬네일 생성, 문서 처리, 메타데이터 추출, 동영상 트랜스코딩

### 2.4 Amazon EventBridge

- **용도**: 이벤트 기반 아키텍처의 중심 이벤트 버스
- **특징**: 고급 필터링, 다양한 AWS 서비스로 라우팅
- **사용 사례**: 복잡한 워크플로우, 다중 서비스 오케스트레이션, 서드파티 통합

**알아두세요!** 📝 모든 S3 이벤트는 이제 기본적으로 EventBridge로도 전송될 수 있습니다. 이 기능은 S3 버킷 설정에서 활성화해야 합니다.

## 3. 필요한 IAM 권한 설정 🔐

S3 이벤트가 다른 서비스로 알림을 보내려면 적절한 권한이 필요합니다. 이는 **리소스 기반 정책**을 통해 구성됩니다:

![[스크린샷 2025-05-08 오후 8.34.48.png]]

### 3.1 SNS 토픽에 대한 리소스 정책

json

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:region:account-id:topic-name",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:*:*:bucket-name"
        }
      }
    }
  ]
}
```

### 3.2 SQS 대기열에 대한 리소스 정책

json

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SQS:SendMessage",
      "Resource": "arn:aws:sqs:region:account-id:queue-name",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:*:*:bucket-name"
        }
      }
    }
  ]
}
```

### 3.3 Lambda 함수에 대한 리소스 정책

json

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:region:account-id:function:function-name",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:*:*:bucket-name"
        }
      }
    }
  ]
}
```

**중요!** ⚠️ S3 이벤트 알림에서는 ==**IAM 역할이 아닌 리소스 기반 정책을**== 사용합니다. 이는 대상 서비스(SNS, SQS, Lambda)가 S3로부터 직접 호출을 받기 때문입니다.

## 4. 실제 사용 사례 및 구현 예시 🛠️

### 4.1 이미지 업로드 시 섬네일 자동 생성

**요구 사항**:

- S3 버킷에 이미지가 업로드되면 자동으로 섬네일 생성
- 원본 이미지와 별도 폴더에 섬네일 저장

**구현 단계**:

1. Lambda 함수 생성 (이미지 처리 로직 포함)
2. Lambda 리소스 정책 설정
3. S3 이벤트 알림 구성:
    - 이벤트 유형: `s3:ObjectCreated:*`
    - 접두사: `uploads/images/`
    - 접미사: `.jpg`, `.png`, `.jpeg`
    - 대상: Lambda 함수

python

```python
# Lambda 함수 예시 (Python)
import boto3
import PIL from Image
import io

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # 이벤트에서 버킷과 키 정보 추출
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # 원본 이미지 다운로드
    response = s3_client.get_object(Bucket=bucket, Key=key)
    image_content = response['Body'].read()
    
    # 이미지 처리
    image = Image.open(io.BytesIO(image_content))
    image.thumbnail((100, 100))
    buffer = io.BytesIO()
    image.save(buffer, format=image.format)
    buffer.seek(0)
    
    # 섬네일 저장
    thumbnail_key = key.replace('uploads/images/', 'thumbnails/')
    s3_client.put_object(
        Bucket=bucket,
        Key=thumbnail_key,
        Body=buffer,
        ContentType=response['ContentType']
    )
    
    return {
        'statusCode': 200,
        'body': f'Thumbnail created at {thumbnail_key}'
    }
```

### 4.2 대용량 파일 업로드 완료 시 처리 워크플로우 시작

**요구 사항**:

- 대용량 파일(.zip, .csv)이 업로드되면 처리 워크플로우 시작
- 처리 단계: 검증 → 변환 → 데이터베이스 적재

**구현 단계**:

1. SQS 대기열 생성
2. SQS 리소스 정책 설정
3. 처리 워커 애플리케이션 구축
4. S3 이벤트 알림 구성:
    - 이벤트 유형: `s3:ObjectCreated:CompleteMultipartUpload`
    - 접미사: `.zip`, `.csv`
    - 대상: SQS 대기열

### 4.3 규정 준수 파일 보관 및 알림

**요구 사항**:

- 중요 문서가 삭제되면 관리자에게 알림
- 문서 삭제 기록 유지

**구현 단계**:

1. SNS 토픽 생성 및 관리자 이메일 구독
2. SNS 리소스 정책 설정
3. S3 이벤트 알림 구성:
    - 이벤트 유형: `s3:ObjectRemoved:*`
    - 접두사: `compliance/documents/`
    - 대상: SNS 토픽

## 5. Amazon EventBridge를 활용한 고급 이벤트 처리 🌐

EventBridge는 S3 이벤트 알림의 능력을 크게 확장시키는 서비스이다:
![[스크린샷 2025-05-08 오후 8.39.53.png]]
### 5.1 EventBridge의 주요 장점

- **고급 필터링**: 객체 메타데이터, 크기, 태그 등 더 많은 속성으로 필터링
- **다중 대상**: 하나의 이벤트를 여러 AWS 서비스로 동시에 라우팅
- **추가 대상 지원**: Step Functions, Kinesis, Firehose 등 지원
- **이벤트 아카이빙**: 이벤트 이력 보존 및 재생 가능
- **이벤트 중계**: 교차 계정, 교차 리전 이벤트 전달
- **안정적인 전달**: 이벤트 전달 보장 메커니즘

### 5.2 EventBridge 활성화 방법

1. S3 버킷 설정에서 "Amazon EventBridge에 알림 전송" 옵션 활성화
2. EventBridge 콘솔에서 S3 이벤트에 대한 규칙 생성
3. 이벤트 패턴 및 대상 지정

### 5.3 EventBridge 이벤트 패턴 예시

json

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["my-bucket"]
    },
    "object": {
      "key": [{
        "prefix": "images/"
      }],
      "size": [{
        "numeric": [">", 5242880]
      }]
    }
  }
}
```

위 패턴은 "my-bucket"의 "images/" 폴더에 5MB보다 큰 객체가 생성될 때 이벤트를 트리거합니다.

## 6. 이벤트 알림 설계 시 고려 사항 ⚖️

### 6.1 성능 및 확장성

- **전송 지연**: 이벤트는 일반적으로 몇 초 이내에 전달되지만, 간혹 몇 분 소요 가능
- **최소 한 번 전달**: 드물게 동일 이벤트가 여러 번 전달될 수 있음
- **순서 보장 없음**: 이벤트 발생 순서와 전달 순서가 다를 수 있음

### 6.2 비용 최적화

- 필요한 이벤트만 필터링하여 불필요한 처리 줄이기
- Lambda 함수 실행 시간 최적화
- EventBridge 규칙 아카이빙 비용 고려

### 6.3 오류 처리 전략

- 실패한 이벤트 처리를 위한 데드 레터 큐(DLQ) 설정
- 멱등성 처리 로직 구현 (동일 이벤트 중복 처리 방지)
- 모니터링 및 경보 설정

## 7. 요약: S3 이벤트 알림의 핵심 포인트 📌

1. **자동화 기반**: S3에서 발생하는 이벤트에 자동으로 대응
2. **다양한 이벤트 유형**: 객체 생성, 삭제, 복원, 복제 등 다양한 이벤트 감지
3. **유연한 필터링**: 접두사, 접미사로 특정 객체에 대한 이벤트만 처리
4. **다중 대상 지원**: SNS, SQS, Lambda, EventBridge 등 다양한 대상으로 전송
5. **권한 모델**: 리소스 기반 정책을 통한 서비스 간 통신 허용
6. **고급 처리**: EventBridge를 통해 더욱 강력한 이벤트 처리 워크플로우 구현

S3 이벤트 알림을 활용하면 스토리지 이벤트에 기반한 완전 자동화된 워크플로우를 구축할 수 있다. 새로운 파일 업로드에 응답하여 처리하거나, 데이터 분석 파이프라인을 트리거하거나, 사용자에게 중요한 변경 사항을 알리는 등 다양한 시나리오에 적용 가능하다


# Amazon S3 성능 최적화: 고성능 스토리지의 비밀 🚀

S3는 기본적으로 뛰어난 성능을 제공하지만, 특정 기법을 활용하면 더 높은 성능을 이끌어낼 수 있다. 
## 1. S3의 기본 성능 특성 📊

Amazon S3는 기본적으로 높은 성능을 제공하도록 설계되었다:

### 1.1 핵심 성능 지표

- **지연 시간**: 첫 번째 바이트까지 100-200밀리초(매우 빠른 응답 시간)
- **자동 스케일링**: 높은 요청 처리량에 맞춰 자동으로 확장

### 1.2 요청 처리량 제한

S3는 접두사(prefix)당 다음과 같은 처리량을 제공합니다:

- **쓰기 작업**: 초당 3,500개 요청 (PUT, COPY, POST, DELETE)
- **읽기 작업**: 초당 5,500개 요청 (GET, HEAD)

**중요 개념!** 🔑 여기서 "접두사"란 버킷 내 객체 키의 경로 부분을 의미합니다. 예를 들어, `s3://my-bucket/folder1/subfolder1/file.txt`에서 접두사는 `/folder1/subfolder1/`입니다.

## 2. 접두사(Prefix)의 개념 이해하기 🗂️

접두사는 S3 성능을 이해하는 데 매우 중요한 개념입니다:

### 2.1 접두사 예시

아래와 같은 4개의 객체가 있다고 가정해 보자:

1. `s3://my-bucket/folder1/sub1/file.txt`
    - 접두사: `/folder1/sub1/`
2. `s3://my-bucket/folder1/sub2/file.txt`
    - 접두사: `/folder1/sub2/`
3. `s3://my-bucket/folder2/sub1/file.txt`
    - 접두사: `/folder2/sub1/`
4. `s3://my-bucket/folder2/sub2/file.txt`
    - 접두사: `/folder2/sub2/`

이 예시에서 우리는 4개의 서로 다른 접두사를 가지고 있다.

### 2.2 접두사와 성능의 관계

- 각 접두사는 독립적으로 초당 3,500개의 쓰기 요청과 5,500개의 읽기 요청을 처리할 수 있다
- 위 예시에서 4개의 접두사에 읽기 요청을 균등하게 분산하면 초당 최대 22,000개(5,500 × 4)의 읽기 요청을 처리할 수 있게 된다
- 버킷 내 접두사 수에는 제한이 없으므로, 충분한 접두사를 사용하면 매우 높은 처리량을 달성할 수 있는 것이다.

**실무 팁!** 💡 매우 높은 처리량이 필요한 애플리케이션의 경우, 객체 키 설계 시 다양한 접두사를 활용하여 병렬 처리 능력을 극대화 해보자 

## 3. 업로드 성능 최적화 기법 ⬆️

### 3.1 멀티파트 업로드(Multipart Upload)

멀티파트 업로드는 큰 파일을 여러 부분으로 나누어 병렬로 업로드하는 기능이다:

- **적용 대상**:
    - 100MB 이상 파일에 권장
    - 5GB 이상 파일에 필수
- **작동 방식**:
    1. 파일을 여러 부분(part)으로 분할
    2. 각 부분을 병렬로 업로드
    3. 모든 부분이 업로드되면 S3에서 자동으로 완전한 객체로 결합
- **주요 이점**:
    - 네트워크 대역폭 최대 활용
    - 업로드 속도 향상
    - 네트워크 문제 발생 시 영향받은 부분만 재시도 가능
    - 업로드 중단 및 재개 지원

**구현 예시** (AWS CLI):

bash

```bash
# 멀티파트 업로드 시작
aws s3api create-multipart-upload --bucket my-bucket --key large-file.zip

# 각 부분 업로드 (병렬로 실행 가능)
aws s3api upload-part --bucket my-bucket --key large-file.zip --part-number 1 --upload-id UPLOAD_ID --body part1

# 멀티파트 업로드 완료
aws s3api complete-multipart-upload --bucket my-bucket --key large-file.zip --upload-id UPLOAD_ID --multipart-upload "Parts=[{PartNumber=1,ETag=...}]"
```

### 3.2 S3 전송 가속화(Transfer Acceleration)

S3 전송 가속화는 AWS의 글로벌 엣지 네트워크를 활용하여 전송 속도를 향상시키는 기능이다:

- **작동 방식**:
    1. 파일을 가장 가까운 AWS 엣지 로케이션으로 전송
    2. 파일은 AWS의 최적화된 네트워크 경로를 통해 대상 S3 버킷으로 이동
- **주요 이점**:
    - 장거리 전송에서 특히 효과적
    - 공용 인터넷 사용 최소화
    - AWS 프라이빗 네트워크 활용 최대화
    - 멀티파트 업로드와 함께 사용 가능
- **사용 사례**:
    - 전 세계에 분산된 사용자가 동일한 버킷에 업로드하는 경우
    - 지리적으로 멀리 떨어진 리전 간 데이터 전송
    - 네트워크 상태가 불안정한 환경에서의 업로드

**실제 예시**: 미국에서 호주의 S3 버킷으로 파일을 업로드하는 경우:

1. 파일은 미국의 가장 가까운 엣지 로케이션으로 업로드 (공용 인터넷 짧은 거리 이동)
2. 엣지 로케이션에서 호주의 S3 버킷으로 AWS 프라이빗 네트워크를 통해 고속 전송

**설정 방법** (AWS CLI):

bash

```bash
# 버킷에서 전송 가속화 활성화
aws s3api put-bucket-accelerate-configuration --bucket my-bucket --accelerate-configuration Status=Enabled

# 전송 가속화를 사용하여 업로드
aws s3 cp large-file.zip s3://my-bucket/ --endpoint-url https://my-bucket.s3-accelerate.amazonaws.com
```

## 4. 다운로드 성능 최적화 기법 ⬇️

### 4.1 바이트 범위 가져오기(Range GET)

바이트 범위 가져오기는 객체의 특정 부분만 요청하는 기능입니다:

- **작동 방식**:
    - HTTP Range 헤더를 사용하여 객체의 특정 바이트 범위만 요청
    - 여러 범위를 병렬로 요청 가능
- **활용 방법**:
    1. **병렬 다운로드**:
        - 큰 파일을 여러 작은 바이트 범위로 나누어 병렬로 다운로드
        - 각 범위는 별도의 요청으로 처리되어 총 다운로드 속도 향상
    2. **부분 검색**:
        - 파일의 특정 부분만 필요한 경우 해당 부분만 검색
        - 예: 큰 로그 파일에서 헤더만 확인하거나, 동영상의 특정 세그먼트만 재생
- **주요 이점**:
    - 네트워크 대역폭 최대 활용
    - 다운로드 속도 향상
    - 필요한 데이터만 가져와 효율성 증가
    - 장애 복원력 향상 (실패한 범위만 재시도 가능)

**구현 예시** (AWS CLI):

bash

```bash
# 파일의 처음 1MB만 다운로드
aws s3api get-object --bucket my-bucket --key large-file.mp4 --range "bytes=0-1048575" first-mb.mp4

# 파일의 특정 바이트 범위만 다운로드
aws s3api get-object --bucket my-bucket --key large-file.mp4 --range "bytes=10485760-20971519" segment.mp4
```

**실제 사용 사례**:

- **헤더 분석**: 파일의 처음 몇 바이트만 가져와 메타데이터/헤더 정보 분석
- **스트리밍 미디어**: 동영상 플레이어에서 사용자가 특정 위치로 이동할 때 해당 부분만 다운로드
- **대용량 데이터셋 처리**: 대용량 데이터 파일의 특정 부분만 처리해야 할 때

## 5. 고급 성능 최적화 전략 🔧

### 5.1 키 이름 설계를 통한 성능 최적화

S3의 요청 분산은 객체 키 이름에 기반하므로, 키 이름 설계가 성능에 큰 영향을 미칠 수 있습니다:

- **순차적 이름 지양**:
    - `2023-01-01_log.txt`, `2023-01-02_log.txt`와 같은 순차적 이름은 같은 파티션에 저장될 가능성이 높음
    - 이로 인해 특정 파티션에 부하 집중 가능성
- **접두사 다양화**:
    - 접두사를 다양하게 하여 더 많은 파티션에 걸쳐 데이터 분산
    - 예: 날짜를 역순으로 (`01-01-2023` 대신 `2023-01-01`) 또는 해시 접두사 추가
- **권장 키 명명 패턴**:
    
    ```
    [해시 또는 랜덤값]/[날짜]/[항목 유형]/[파일명]
    ```
    

**실무 팁!** 🌟 2018년 7월 이후, Amazon S3는 성능 향상을 위해 파티셔닝 스키마를 개선했습니다. 순차적 키 이름도 이전보다 좋은 성능을 보이지만, 최고의 성능을 위해서는 여전히 키 이름 다양화가 권장됩니다.

### 5.2 S3 Select 활용

S3 Select를 사용하면 전체 객체를 다운로드하지 않고 SQL 쿼리를 통해 필요한 데이터만 검색할 수 있습니다:

- **작동 방식**:
    - 서버 측에서 필터링을 수행하여 필요한 데이터만 반환
    - SQL과 유사한 쿼리 언어 사용
- **지원 파일 형식**:
    - CSV, JSON, Parquet
- **이점**:
    - 전송 데이터 양 감소
    - 애플리케이션 처리 시간 단축
    - 네트워크 대역폭 절약

**구현 예시** (AWS CLI):

bash

```bash
aws s3api select-object-content \
    --bucket my-bucket \
    --key large-dataset.csv \
    --expression "SELECT * FROM s3object s WHERE s.\"age\" > 21" \
    --expression-type 'SQL' \
    --input-serialization '{"CSV": {"FileHeaderInfo": "USE"}}' \
    --output-serialization '{"CSV": {}}' \
    filtered-output.csv
```

## 6. 성능 모니터링 및 문제 해결 📈

### 6.1 성능 모니터링

S3 성능을 모니터링하는 주요 도구와 지표:

- **Amazon CloudWatch**:
    - `FirstByteLatency`: 첫 바이트까지의 지연 시간
    - `TotalRequestLatency`: 총 요청 지연 시간
    - `4xxErrors` 및 `5xxErrors`: 오류 비율
- **S3 Request Metrics**:
    - 버킷, 접두사 또는 태그별 상세 지표 제공
    - 추가 비용 발생 가능
- **S3 Storage Lens**:
    - 스토리지 사용량 및 활동 분석
    - 최적화 권장 사항 제공

### 6.2 일반적인 성능 문제 및 해결책

| 문제       | 가능한 원인          | 해결책                         |
| -------- | --------------- | --------------------------- |
| 높은 지연 시간 | 지리적 거리          | 전송 가속화 활성화, 거리가 가까운 리전 사용   |
| 느린 업로드   | 큰 파일 크기         | 멀티파트 업로드 사용                 |
| 병목 현상    | 소수의 접두사에 트래픽 집중 | 키 이름 다양화, 접두사 분산            |
| 처리량 제한   | 단일 접두사에 과도한 요청  | 요청을 여러 접두사로 분산              |
| 처리 지연    | 불필요한 데이터 다운로드   | S3 Select 또는 바이트 범위 가져오기 사용 |

## 7. 요약: S3 성능 최적화의 핵심 포인트 📌

1. **기본 성능 이해**:
    - 접두사당 초당 3,500개 쓰기 및 5,500개 읽기 요청
    - 접두사를 다양화하여 성능 향상
2. **업로드 최적화**:
    - 100MB 이상 파일에는 멀티파트 업로드 사용
    - 장거리 전송에는 S3 전송 가속화 활성화
3. **다운로드 최적화**:
    - 큰 파일은 바이트 범위 가져오기로 병렬화
    - 필요한 데이터만 검색하여 효율성 향상
4. **키 이름 설계**:
    - 순차적 접두사 지양
    - 다양한 접두사 활용으로 부하 분산
5. **고급 기능 활용**:
    - S3 Select로 서버 측 필터링
    - CloudWatch로 성능 모니터링 및 병목 식별

Amazon S3는 기본적으로 뛰어난 성능을 제공하지만, 이러한 최적화 기법을 적용하면 대규모 워크로드에서도 탁월한 성능을 발휘할 수 있습니다. 특히 대용량 파일 처리, 높은 처리량 요구 사항, 글로벌 액세스 패턴을 가진 애플리케이션에서 이러한 기법들이 큰 차이를 만들 수 있습니다! 🚀



# Amazon S3 Batch Operations: 대규모 객체 작업의 효율적인 관리 🧰

수백만 개의 객체를 처리해야 할 때 개별 API 호출로는 비효율적이고 시간이 많이 소요된다. S3 Batch Operations는 이러한 대규모 작업을 단일 요청으로 처리할 수 있는 강력한 솔루션을 제공한다.

## 1. S3 Batch Operations 개요 📋

### 1.1 S3 Batch Operations란?

S3 Batch Operations는 단일 요청으로 기존 S3 객체에 대해 대규모 작업을 수행할 수 있게 해주는 서비스이다. 수백만, 심지어 수십억 개의 객체에 대해 미리 정의된 작업을 실행할 수 있게 해준다.

### 1.2 주요 특징

- **대규모 처리**: 단일 작업으로 수백만 개의 객체 처리
- **작업 관리**: 작업 진행 상황 추적, 완료 알림, 보고서 생성
- **오류 처리**: 자동 재시도 및 오류 보고
- **우선 순위 지정**: 작업 우선 순위 설정 가능
- **감사 및 보고**: 상세한 완료 보고서 생성

### 1.3 작업 구성 요소

S3 Batch Operations 작업은 다음과 같은 요소로 구성된다:

- **매니페스트(객체 목록)**: 처리할 객체 목록
- **작업 유형**: 수행할 작업(복사, 태그 지정 등)
- **매개변수**: 작업에 필요한 추가 정보
- **우선 순위**: 여러 작업 간의 우선 순위
- **역할**: 작업 실행에 필요한 IAM 역할

## 2. 지원되는 작업 유형 🛠️

S3 Batch Operations는 다양한 작업 유형을 지원한다:

### 2.1 S3 객체 복사

여러 객체를 한 버킷에서 다른 버킷으로 복사할 수 있다.

**주요 기능**:

- 여러 리전 간 복사 지원
- 스토리지 클래스 변경 가능
- 객체 메타데이터, 태그 수정 가능
- 서버 측 암호화 적용 가능

**사용 사례**:

- 데이터 마이그레이션
- 스토리지 계층화
- 백업 생성

### 2.2 객체 속성 및 메타데이터 관리

객체의 다양한 속성을 일괄적으로 변경할 수 있다.

**가능한 작업**:

- 객체 태그 설정/삭제
- 액세스 제어 목록(ACL) 변경
- 객체 메타데이터 수정
- 스토리지 클래스 변경

**사용 사례**:

- 데이터 분류 및 태깅
- 액세스 권한 일괄 업데이트
- 규정 준수를 위한 메타데이터 추가

### 2.3 S3 객체 암호화

암호화되지 않은 객체를 암호화하거나 암호화 설정을 변경할 수 있다.

**주요 기능**:

- SSE-S3, SSE-KMS, SSE-C 암호화 설정 가능
- 기존 암호화 방식 변경 지원

**사용 사례**:

- 보안 규정 준수
- 암호화되지 않은 레거시 객체 보호
- 암호화 키 교체

### 2.4 S3 Glacier 복원

Glacier나 Glacier Deep Archive에 저장된 여러 객체를 일괄적으로 복원시킬 수도 있다.

**주요 기능**:

- 복원 기간 지정 가능
- 검색 계층 선택 가능
- 복원 우선 순위 설정 가능

**사용 사례**:

- 대규모 아카이브 데이터 분석
- 비상 시 데이터 복구
- 규정 준수를 위한 기록 검토

### 2.5 Lambda 함수 호출

사용자 정의 Lambda 함수를 호출하여 객체에 대한 복잡한 처리를 수행할 수도 있다.

**주요 기능**:

- 객체당 Lambda 함수 호출
- 사용자 정의 로직 실행 가능
- 복잡한 변환 작업 지원

**사용 사례**:

- 이미지 처리(크기 조정, 워터마크 등)
- 콘텐츠 분석 및 메타데이터 추출
- 데이터 변환 및 정규화

## 3. 객체 목록 생성 방법 📝

S3 Batch Operations에 전달할 객체 목록(매니페스트)을 생성하는 두 가지 주요 방법이 있다:
![[스크린샷 2025-05-08 오후 11.16.36.png]]
### 3.1 S3 Inventory 활용

S3 Inventory는 버킷 내 **객체의 목록**과 **메타데이터**를 정기적으로 생성하는 서비스이다.

**S3 Inventory 과정**:

1. 버킷에 대한 S3 Inventory 구성 설정
2. 정기적(일별 또는 주별)으로 CSV/ORC/Parquet 형식의 보고서 생성
3. 생성된 인벤토리 보고서를 S3 Batch Operations의 **입력**으로 사용

**주요 장점**:

- 정기적으로 자동 생성
- 메타데이터(크기, 마지막 수정 날짜, 스토리지 클래스, 암호화 상태 등) 포함
- 대규모 버킷에서도 효율적

### 3.2 S3 Select로 필터링

S3 Select를 사용하여 인벤토리 보고서나 다른 객체 목록에서 특정 조건을 충족하는 객체만 필터링하는 것이다.

**S3 Select 사용 예시**:

sql

```sql
SELECT * FROM s3object s WHERE s.Encrypted = 'false'
```

암호화되지 않은 객체만 선택하는 쿼리이다.

**주요 장점**:

- 특정 조건에 맞는 객체만 처리
- 복잡한 필터링 가능
- 처리 시간 및 비용 절감

### 3.3 사용자 정의 매니페스트 생성

CSV 파일을 직접 생성하여 매니페스트(처리할 객체 목록)로 사용할 수도 있다.

**CSV 형식 요구사항**:

- 각 행은 하나의 객체 정보 포함
- 기본 형식: `버킷,키` 또는 `버킷,키,버전ID`

**예시 CSV**:

```
my-bucket,photos/2023/image1.jpg
my-bucket,photos/2023/image2.jpg
my-bucket,documents/report.pdf
```

**주요 장점**:

- 완전한 유연성 제공
- 다양한 소스의 객체 포함 가능
- 특정 객체만 정확히 지정 가능

## 4. S3 Batch Operations 워크플로우 🔄

S3 Batch Operations의 일반적인 워크플로우는 다음과 같다:

### 4.1 기본 워크플로우

1. **매니페스트 준비**:
    - S3 Inventory 설정 및 보고서 생성
    - S3 Select로 필요한 객체 필터링
    - 또는 사용자 정의 CSV 매니페스트 생성
2. **배치 작업 생성**:
    - 작업 유형 선택
    - 매개변수 구성
    - IAM 역할 지정
    - 완료 보고서 옵션 설정
3. **작업 실행 및 모니터링**:
    - 작업 진행 상황 추적
    - 실패한 작업 분석
    - 필요시 작업 일시 중지 또는 취소
4. **결과 분석**:
    - 완료 보고서 검토
    - 성공 및 실패 작업 확인
    - 필요시 후속 작업 계획

### 4.2 실제 예시: 암호화되지 않은 객체 일괄 암호화

1. **암호화되지 않은 객체 식별**:
    
    ```
    # S3 Inventory 설정
    - 암호화 상태 포함 설정
    - 일별 보고서 생성
    
    # S3 Select로 암호화되지 않은 객체 필터링
    SELECT * FROM s3object s WHERE s.Encrypted = 'false'
    ```
    
2. **암호화 배치 작업 생성**:
    
    ```
    # 암호화 작업 구성
    - 작업 유형: 객체 복사
    - 대상: 동일 위치
    - 암호화: SSE-KMS
    - KMS 키 지정
    ```
    
3. **작업 실행 및 모니터링**:
    - Amazon CloudWatch로 진행 상황 모니터링
    - 완료 시 SNS 알림 수신
4. **결과 검증**:
    - 완료 보고서 분석
    - 모든 객체가 제대로 암호화되었는지 확인


## 5. 요약: S3 Batch Operations의 핵심 포인트 📌

1. **대규모 작업 자동화**:
    - 수백만 개의 객체에 대한 작업을 단일 요청으로 처리
    - 수동 스크립팅의 복잡성 제거
2. **다양한 작업 지원**:
    - 복사, 태그 지정, 암호화, ACL 변경, Glacier 복원 등
    - Lambda 함수를 통한 사용자 정의 작업 지원
3. **효율적인 객체 선택**:
    - S3 Inventory를 통한 객체 목록 생성
    - S3 Select를 통한 정밀한 필터링
4. **강력한 작업 관리**:
    - 진행 상황 추적 및 모니터링
    - 실패한 작업 자동 재시도
    - 상세한 완료 보고서 생성
5. **주요 사용 사례**:
    - 일괄 암호화 및 보안 강화
    - 데이터 마이그레이션 및 아카이브
    - 규정 준수 및 거버넌스 적용

S3 Batch Operations는 대규모 S3 환경에서 관리 부담을 크게 줄이고, 반복적인 작업을 자동화하며, 일관된 방식으로 대량의 객체를 처리할 수 있게 해준다 특히 규정 준수 요구 사항, 데이터 마이그레이션, 보안 강화와 같은 중요한 작업에서 유용하다.


Storage Lens
![[스크린샷 2025-05-08 오후 11.24.59.png]]