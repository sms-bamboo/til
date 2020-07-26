# KMS
KMS는 데이터 암호화에 사용할 암호화 키를 쉽게 생성하고 삭제할 수 있게 해주는 관리형 서비스이다.

## 1. 관리형 Key
#### AWS 관리형 Key
AWS 에서 관리하는 암호화 키로, 변경하거나 삭제할 수 없다.
  
#### 고객 관리형 Key (CMK)
사용자가 생성하여 소유하고 관리하는 키로, 정책 설정, 권한 부여 등의 설정을 직접 할 수 있다.

## 2. CMK 교체
#### 자동 교체
* 교체 일정은 365 일으로, 이는 교체할 수 없다.  
* 키가 교체될 때 CMK의 속성(정책, 권한 등) 은 변경되지 않는다.
* 키를 교체하면 암호화 작업에 사용되는 CMK의 백업 키만 변경된다.
* 교체된 키 구성 요소는 삭제되지 않고 저장되기 때문에, 교체 이후에도 추가 작업이 필요하지 않다.
  
#### 수동 교체
* 새로운 키를 생성하여 기존의 키를 교체하는 방식이다.
* 교체 일정을 제어하고자 할 때 사용한다.  
* 키가 교체될 때 CMK의 속성(정책, 권한 등) 은 변경되지 않는다.
* 새로운 키가 생성된 것이기 때문에, 어플리케이션에서 키에 대한 참조를 변경해주어야 한다.
* 이 경우 Alias 를 사용하여 기존 키와 연결할 수 있다.

## 3. Key 정책
CMK 에 대한 액세스를 관리하기 위해 정책을 사용한다. (기존 다른 정책들과 유사한 구조)  
#### [예시]
````
{
  "Version": "2012-10-17",
  "Id": "key-consolepolicy-2",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow attachment of persistent resources",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/CMKUser",
        "arn:aws:iam::111122223333:role/CMKRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": "*",
      "Condition": {"Bool": {"kms:GrantIsForAWSResource": "true"}}
    }
  ]
}
````

## 4. 권한 부여
권한 부여 또한 Key 정책과 동일하게 CMK 액세스를 허용하기 위해 사용하는 방식이다.  
다만 권한 부여는 프로그래밍 방식을 사용하며, 편하게 권한을 부여하고 취소하기 위한 임시 권한에 가깝다.

#### [AWS CLI 예시]
````
$ aws kms create-grant \
    --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
    --grantee-principal arn:aws:iam::111122223333:user/exampleUser \
    --operations Decrypt \
    --retiring-principal arn:aws:iam::111122223333:role/adminRole \
    --constraints EncryptionContextSubset={Department=IT}
````
#### [AWS SDK JAVA 예시]
````
String keyId = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
String granteePrincipal = "arn:aws:iam::111122223333:user/Alice";
String operation = GrantOperation.GenerateDataKey.toString();

CreateGrantRequest request = new CreateGrantRequest()
    .withKeyId(keyId)
    .withGranteePrincipal(granteePrincipal)
    .withOperations(operation);

CreateGrantResult result = kmsClient.createGrant(request);
````

## 5. 삭제
바로 삭제가 불가하며, 7~30 일 사이의 삭제 대기 기간을 예약해야 한다.

* 기본 대기 기간 값은 30일이다.
* 삭제 대기 중인 CMK는 암호화 작업에 사용할 수 없다.
* 삭제 대기 중인 CMK의 백업 키는 교체되지 않는다.
