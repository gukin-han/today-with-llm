# MySQL 인덱스와 스키마 관리

MySQL의 인덱스 및 스키마를 관리하는 방법을 정리한 문서입니다.

## 환경별 DB 인덱스 불일치 점검

### 인덱스 전수 조사 쿼리

* dev / staging / prod에서 모두 동일한 쿼리로 인덱스 리스트를 뽑는다.
* MySQL 기준 예시:

```sql
SELECT 
    TABLE_SCHEMA,
    TABLE_NAME,
    INDEX_NAME,
    NON_UNIQUE,
    SEQ_IN_INDEX,
    COLUMN_NAME,
    COLLATION,
    INDEX_TYPE,
    SUB_PART
FROM information_schema.STATISTICS
WHERE TABLE_SCHEMA = 'your_db_name'
ORDER BY TABLE_NAME, INDEX_NAME, SEQ_IN_INDEX;
```

* 각 환경에서 CSV로 export하여 `indexes_dev.csv`, `indexes_stg.csv`, `indexes_prd.csv` 형태로 저장해 비교한다.

### 차이 비교 & 문서화

* 기준(Source of Truth)을 보통 **prod 스키마**로 잡는다.
* 비교 포인트:

  * prod에는 있는데 dev/stg에는 없는 인덱스
  * dev/stg에는 있는데 prod에는 없는 인덱스 (실험용/불필요 가능성)
  * 같은 이름인데 컬럼 순서, 유니크 여부가 다른 인덱스
* 문서화 예시:

  * `인덱스명 / 테이블 / prod 정의 / dev 정의 / stg 정의 / 액션(추가·삭제·재생성) / 비고` 형태의 표로 관리.

### 인덱스를 다시 맞추는 전략

* 누락 인덱스: prod 정의를 기준으로 dev/stg에 `ADD INDEX` 적용.
* 불필요 인덱스: 사용 여부·쿼리 플랜 확인 후 `DROP INDEX` 고려.
* 정의 차이: `DROP INDEX + ADD INDEX`로 prod와 동일하게 재정의.
* 모든 변경은 **수동 실행이 아니라 마이그레이션 스크립트(예: SQL 파일, 마이그레이션 도구)**로 관리한다.

## ALTER TABLE 쿼리 해석

### ALTER TABLE … CHANGE COLUMN …

```sql
ALTER TABLE TM_LEAVE_CRT 
CHANGE COLUMN IS_CLOSED IS_CLOSED 
    BOOLEAN 
    NULL 
    DEFAULT 0 
    COMMENT '마감여부';
```

* `ALTER TABLE TM_LEAVE_CRT`

  * 해당 테이블의 스키마를 수정.
* `CHANGE COLUMN IS_CLOSED IS_CLOSED`

  * 기존 컬럼 이름을 유지하면서 정의를 변경(MySQL 문법상 기존/새 이름을 모두 적어야 함).
* `BOOLEAN NULL DEFAULT 0`

  * 타입을 BOOLEAN(TINYINT(1)과 유사)로 지정.
  * NULL 허용.
  * 기본값 0.
* `COMMENT '마감여부'`

  * 컬럼 설명 메타데이터 추가.

→ 실제 의미: **기존 IS_CLOSED 컬럼을 “BOOLEAN NULL DEFAULT 0, 마감여부”로 재정의하는 쿼리**.
