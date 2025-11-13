# AWS S3

AWS S3의 핵심 개념(Bucket, ACL 등)을 정리한 문서입니다.

## Bucket과 ACL 용어

### Bucket

* S3에서 객체(object)를 저장하는 **최상위 컨테이너**.
* 전 세계에서 유일한 이름을 가져야 하고, 일종의 “논리적인 스토리지 단위” 역할을 한다.
* 예:

  * `app-prod-bucket`, `logs-archive-bucket` 등(실제 이름은 서비스별로 구성).

### ACL (Access Control List)

* 버킷/객체 단위의 **권한 제어 방식 중 하나**.
* “누가(read/write) 무엇을 할 수 있는지”를 객체/버킷 수준에서 정의하는 규칙 집합.
* 현재는 버킷 정책, IAM 정책 기반 제어가 선호되며, ACL은 필요한 경우에만 제한적으로 활용되는 편이다.
