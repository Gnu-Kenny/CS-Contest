1. **대부분의 RDB는 B-Tree (또는 B+Tree)를 인덱스로 사용합니다. 왜 인덱스 자료구조로 트리를 선택했을까요? 또, 왜 RBT나 AVL Tree는 선택받지 못했을까요?**

<br>

DB는 Memory(RAM)를 거쳐 결과적으로 Secondary Storage(SSD, HDD 등)에 데이터를 저장하게 됩니다. 성능적인 측면에서 Secondary Storage에는 최대한 적게 접근하는 것이 좋기 때문에 DB는 연관된 데이터를 모아서 저장하면 효율적으로 읽고 쓸 수 있을 것입니다.

이러한 측면에서 RBT나 AVL Tree의 Node는 하나의 데이터 밖에 가질 수 없는 반면 B-Tree의 Node는 여러개의 연관된 데이터를 저장할 수 있습니다. 이는 트리의 레벨을 줄여 탐색 범위를 빠르게 좁힐 수 있는 장점을 지니며 데이터를 조회하기 위한 테이블 랜덤 액세스를 줄이는 장점으로 작용합니다.

<br>

---

2. **레코드 수가 아주 많을 것으로 예상되는 대용량 테이블을 어떻게 설계할 것인지 설명해주세요. 또, 해당 테이블에서 어떻게 쿼리 최적화를 할 것인지 설명해주세요.**

OLTP 특정을 지니는 시스템일 경우

정규화를 통해 중복 데이터를 최소화하고 데이터 일관성을 유지하려 노력할 것 입니다. 그리고 반복되는 쿼리가 많을 것이 예상되므로 첫번째로 RDMS가 쿼리 구문 분석시 라이브러리 캐시에서 해당 명령문을 찾아 즉시 실행하도록 쿼리 문법을 통일화할 것입니다. 두번째로 성능을 향상 시키기 위해 결합 인덱스를 통한 커버링 인덱스를 사용하여 테이블 랜덤 액세스를 최소화하도록 튜닝할 것입니다.

<br>

OLAP 특정을 지니는 시스템일 경우

생성일을 기준으로 테이블을 파티셔닝하여 일자별로 데이터를 분리하여 대량 접근시 I/O를 줄일 것 입니다. 그리고 쿼리 최적화의 경우 첫번째로 병렬 쿼리 힌트를 주어 병렬적으로 데이터를 읽어오도록 튜닝할 것입니다. 그리고 materialized view를 구성하여 여러가지 Aggregate Operation 에 대한 실행 시간의 수행속도 향상을 기대할 것입니다.

<br>

---

3. **트랜잭션의 개념을 설명하고, ACID에 대해 최대한 자세히 설명해주세요.**

트랜잭션은 데이터베이스에서 하나의 논리적 기능을 수행하기 위한 작업의 단위를 말하며 여러 개의 쿼리들을 하나로 묶는 단위를 말합니다. 이에 대한 특징에는 원자성, 일관성, 독립성, 지속성이 있으며 이를 한꺼번에 ACID 특징이라고 합니다.

원자성

트랜잭션과 관련된 일이 모두 수행되었거나 되지 않았거나를 보장하는 특징입니다. 예를 들어 트랜잭션을 커밋했는데, 문제가 발생하여 롤백하는 경우 그 이후에 모두 수행되지 않음을 보장하는 것을 말합니다.

일관성

허용된 방식으로만 데이터를 변경해야 하는 것을 의미합니다. 데이터베이스에 기록된 모든 데이터는 여러 가지 조건, 규칙에 따라 유효함을 가져야 합니다.

격리성

격리성은 트랜잭션 수행 시 서로 끼어들지 못하는 것을 말합니다. 복수의 병렬 트랜잭션은 서로 격리되어 마치 순차적으로 실행되는 것처럼 작동되어야 하고, 데이터베이스는 여러 사용자가 같은 데이터에 접근할 수 있어야 합니다.

지속성

성공적으로 수행된 트랜잭션은 영원히 반영되어야 하는 것을 의미합니다. 이는 데이터베이스에 시스템 장애가 발생해도 원래 상태로 복구하는 회복 기능이 있어야 함을 뜻하며, 데이터베이스는 이를 위해 체크섬, 저널링, 롤백 등의 기능을 제공합니다.

<br>
<br>

3-1. **DB에서 직접 동시성을 제어하는 방법과 어플리케이션 레벨에서 동시성을 제어하는 방법이 무엇이 있는지 각각 설명하고, 두 방법을 비교해주세요.**

데이터베이스에서 동시성을 제어하는 방법으로는 Lock을 사용하는 방법이 있습니다. 트랜잭션이 데이터를 업데이트하는 동안 다른 트랜잭션의 접근을 차단합니다.

애플리케이션 레벨에서 동시성을 제어하는 방법으로는 비관적 / 낙관적 방법이 있습니다. 첫번째로 비관적 동시성 제어에서는 트랜잭션을 시작하기 전에 데이터를 읽을 때 해당 데이터를 잠그고, 업데이트할 때까지 다른 트랜잭션의 접근을 막습니다. 이는 구현이 간단하고 데드락의 위험이 낮지만 성능저하가 발생할 수 있습니다. 낙관적 동시성 제어 내 트랜잭션은 데이터를 읽을 때 잠그지 않고, 업데이트할 때 충돌을 감지하고 처리합니다. 충돌이 발생하면 롤백하거나 다시 시도할 수 있습니다.
