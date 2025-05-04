[[Design Secure Architectures]]

IAM이 User가 무엇을 할 수 있는지를 control 하는 것이였다. 
이번에는 AWS 자체에 어떻게 접속할지를 알아보도록 하겠다.

### 🔑 AWS 접근 방법 - 어떻게 들어갈까?

AWS에 접근하는 방법에는 세 가지가 있다.

- 💻 AWS 관리 콘솔(비밀번호 + MFA로 보호): 웹 브라우저로 접근하는 그래픽 인터페이스이다. 우리가 흔히 접속하는 방법
-  AWS 명령줄 인터페이스(CLI): 액세스 키로 보호
- AWS 소프트웨어 개발 키트(SDK): 액세스 키로 보호

액세스 키는 AWS 콘솔을 통해 생성되며, 다음과 같은 특성을 가진다:

- 사용자가 자신의 액세스 키를 관리 <- 생성할 때 한 번 나옴
- 액세스 키는 절대 공유하면 안 된다
- 액세스 키 ID = 사용자 이름
- 비밀 액세스 키 = 비밀번호

#### 액세스 키 예시

액세스 키는 다음과 같은 형태로 제공되는데 CLI, SDK로 접속하려 할 때 필요하다.

- 액세스 키 ID: AKIASK4E37PV4983d6C
- 비밀 액세스 키: AZPN3zojWozWCndIjhB0Unh8239a1bzbzO5fqqkZq


### AWS CLI

AWS CLI(명령줄 인터페이스)는 명령줄에서 명령을 사용하여 AWS 서비스와 상호작용할 수 있는 도구이다

- AWS 서비스의 공개 API에 직접 접근할 수 있어다! 콘솔 화면 없이도 AWS의 모든 기능을 사용할 수 있다는 뜻이다. 
- 리소스 관리를 위한 스크립트를 작성할 수 있다! 반복적인 작업을 자동화하고 싶을 때 정말 유용하다.
- 오픈 소스인 것도 특징이다. [https://github.com/aws/aws-cli에서](https://github.com/aws/aws-cli%EC%97%90%EC%84%9C) 소스 코드를 확인할 수 있다
- AWS 관리 콘솔을 사용하는 대신 명령줄로 작업할 수 있는 대안이다. (근데 나는 어렵더라...)

처음에는 어렵지만 익숙해지면 더 정확하고 효율적으로 제어할 수 있다. "확실히 merit가 있긴 해보인다"

#### AWS CLI 사용 예시

```bash
# S3 버킷 목록 조회하기
aws s3 ls

# EC2 인스턴스 시작하기
aws ec2 run-instances --image-id ami-12345678 --instance-type t2.micro --key-name MyKeyPair

# DynamoDB 테이블에서 데이터 가져오기
aws dynamodb get-item --table-name MyTable --key '{"id":{"S":"123"}}'

# CloudWatch 로그 그룹 생성하기
aws logs create-log-group --log-group-name my-logs

```

이렇게 다양한 AWS 서비스를 명령줄에서 바로 제어할 수 있다 콘솔에서 클릭하고 입력하는 모든 작업을 한 줄의 명령으로 처리할 수 있으나 초기 사용자는 콘솔이 좀 더 편하긴하다

### 📦 AWS SDK란? - 코드로 AWS 다루기

AWS SDK(Software Development Kit)는 애플리케이션 코드 안에서 AWS 서비스를 프로그래밍 방식으로 관리할 수 있게 해주는 도구 모음이다.

어플리케이션 코드 안에서 AWS와 직접 소통하며 필요한 작업을 수행할 수 있게 해준다. 

- 다양한 프로그래밍 언어를 위한 SDK: JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++ 등
- 모바일 SDK: Android, iOS 등
- IoT 디바이스 SDK: 임베디드 C, Arduino

AWS CLI도 Python AWS SDK를 기반으로 만들어졌다.

##### AWS SDK 사용 예시 - 코드로 AWS 다루기
```javascript
// JavaScript(Node.js)에서 S3 버킷 목록 가져오기
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

s3.listBuckets((err, data) => {
  if (err) console.log(err, err.stack);
  else console.log('S3 버킷 목록:', data.Buckets);
});

// Python에서 DynamoDB 항목 가져오기
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('MyTable')
response = table.get_item(Key={'id': '123'})
item = response.get('Item')
print(f'가져온 항목: {item}')

```


### 🎭 IAM 역할(Roles) - 서비스를 위한 임시 권한

![사진](iam5.png)
IAM Role은 AWS 서비스가 User 처럼 작업을 수행할 수 있도록 임시 권한을 부여하는 특별한 방법이다. 
어떤 AWS 서비스가 다른 작업을 수행해야 할 때, 나의 자격 증명을 직접 공유하는 대신 IAM 역할을 통해 안전하게 권한을 부여할 수 있다. 이렇게 하면 자격 증명을 코드에 하드코딩하거나 공유하는 위험한 방법을 피할 수 있게 된다🛡️

가장 흔하게 사용되는 IAM 역할들을 살펴보자 👇

- EC2 인스턴스 역할: EC2 서버가 S3 버킷에 접근하거나 DynamoDB 테이블을 읽는 등의 작업을 수행할 수 있도록 한다.
- Lambda 함수 역할: 서버리스 Lambda 함수가 다른 AWS 서비스와 상호작용할 수 있게 해준다.함수가 CloudWatch에 로그를 쓰거나 S3 파일을 처리하는 등의 작업을 수행할 수 있다 📝
- CloudFormation 역할: 인프라를 자동으로 구축할 때 필요한 권한을 부여한다. 인프라 자동화의 핵심이라고 할 수 있다.


### 🔍 IAM 보안 도구 - 계정 상태 확인하기

AWS는 IAM 설정이 안전한지 확인할 수 있는 유용한 도구들을 제공해주는데 정기적으로 보안 상태를 체크해주는 것이 중요하다. 

- IAM 자격 증명 보고서(계정 수준): 계정 내 모든 사용자와 그들의 자격 증명 상태를 보여주는 보고서이다. MFA가 활성화되었는지, 비밀번호가 언제 마지막으로 변경되었는지 등을 한눈에 확인할 수 있게 된다
- IAM 접근 관리자(사용자 수준): 특정 사용자에게 부여된 서비스 권한과 그 서비스에 마지막으로 접근한 시간을 보여주는데 이 정보를 바탕으로 불필요한 권한을 제거하고 정책 최적화를 진행한다. 

보안은 한 번에 끝나는 게 아니라 계속 관리해야 하는 부분이기 때문에 AWS에서 이런 서비스를 제공하는 것이다. 🔄


### 📝 IAM 모범 사례 

모범 사례 같은 부분들도 시험에 출제 될 수가 있으니까 잘 봐두면 좋을 것 같다. 근데 어찌보면 당연한 것들이라서 인지만 하고 넘어가도록 하자

- 루트 계정은 AWS 계정 설정 이외에는 사용하지 마라
- 한 명의 실제 사용자 = 한 개의 AWS 사용자 계정을 원칙으로 
	- 여러 사람이 하나의 계정을 공유하면 누가 무슨 작업을 했는지 추적하기 어려워지기 때문이다 👤
- 사용자들을 그룹에 할당하고, 그룹에 권한을 부여해라! 
	- 개별 사용자마다 권한을 설정하는 것보다 그룹 단위로 관리하는 게 훨씬 효율적이다 👥
- 강력한 비밀번호 정책을 만들고 적용해라! 
- 다중 인증(MFA)을 사용하고 모든 사용자에게 적용해라! 
- AWS 서비스에 권한을 부여할 때는 역할을 사용해라! 
	- 자격 증명을 직접 공유하는 것보다 역할 위임이 훨씬 안전하다 🎭
- 프로그래밍 방식 접근(CLI/SDK)에는 액세스 키를 사용해라! 
	- Access Key 의 보안이 정말 중요한 것도 리마인드 하자
- IAM 자격 증명 보고서와 IAM 접근 관리자를 사용해 계정을 정기적으로 관찰 해라 

### ✨ IAM 총정리 

- 사용자(Users): 실제 사람에 매핑되며, AWS 콘솔에 로그인할 수 있는 비밀번호를 가짐
- 그룹(Groups): 사용자만 포함할 수 있음
- 정책(Policies): 사용자나 그룹의 권한을 정의하는 JSON 문서
- 역할(Roles): EC2 인스턴스나 AWS 서비스를 위한 것
- 보안(Security): MFA + 비밀번호 정책으로 강화
- AWS CLI: 명령줄에서 AWS 서비스 관리
- AWS SDK: 프로그래밍 언어로 AWS 서비스 활용
- 액세스 키(Access Keys): CLI나 SDK로 AWS에 접근할 때 사용
- 감사(Audit): IAM 자격 증명 보고서 & IAM 접근 관리자

AWS IAM은 클라우드 보안의 기초이며, 이를 잘 이해하고 활용하면 AWS 서비스를 안전하게 사용할 수 있게 된다.
