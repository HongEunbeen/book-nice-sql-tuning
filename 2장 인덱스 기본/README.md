# 2장 인덱스 기본

## 인덱스 구조 및 탐색

### [데이터를 찾는 방법]

데이터베이스 테이블에서 데이터를 찾는 방법은 아래 두 가지가 있습니다.

- 테이블 전체를 스탠한다.
- 인덱스를 이용한다.

테이블 전체 스탠의 경우 튜닝 요소가 많이 없지만, 인덱스와 관련해서는 튜닝 요소가 많기때문에 인덱스는 SQL 튜닝을 공부할 때 가장 먼저 다루어야 합니다.

### [인덱스 튜닝의 두 가지 핵심요소]

인덱스는 큰 테이블에서 소량 데이터를 검색할 때 사용합니다. 

1. 인덱스 스캔 효율화 튜닝
    
    인덱스 스캔 과정에서 발생하는 비효율을 줄입니다.
    
2. 랜덤 액세스 최소화 튜닝
    
    테이블 액세스 횟수를 줄입니다.
    

둘 중 랜덤 액세스 최소화 튜닝이 조금 더 성능에 미치는 영향이 큽니다.

### [SQL 튜닝은 랜덤 I/O와의 전쟁]

DBMS가 제공하는 많은 기능은 느린 랜덤 I/O를 위해 개발되었습니다. 이런 기능들의 본질은 랜덤 I/O를 줄이는데 있습니다.

### [인덱스 구조]

```
인덱스
- 대용량 테이블에서 필요한 데이터만 빠르게 효율적으로 액세스하기 위해 사용하는 오브젝트
```

- 인덱스가 없다면? : 테이블을 처음부터 끝까지 모두 읽어야 합니다.
- 인덱스가 있다면? : 일부만 읽고 멈출 수 있습니다. (범위 스캔)

범위 스캔이 가능한 이유는 인덱스가 키값으로 정렬되어 있기 떄문입니다. 

만약, 인덱스 키 값이 같다면 ROWID로 정렬됩니다.

```
B*Tree
- 나무를 거꾸로 뒤집은 모양으로 뿌리가 위쪽에 있고, 가지를 거쳐 맨 아래 잎사귀가 있습니다.
각 레코드는 하위 블록에 대한 주소값(ROWID)을 가지고, 키값은 하위 블록에 저장된 키값의 범위를 나타냅니다.

LMC
- 루트와 브랜치 블록에는 키값을 갖지 않는 특별한 레코드를 가지며 자식 노드 중 가장 왼쪽 끝
에 위치한 블록을 가리킵니다.

ROWID
- 데이터 블록 주소 + 로우 번호로 구성되어 있으며 이 값을 알면 테이블 레코드를 찾아갈 수 있습니다.

데이터 블록 주소 
- 데이터 파일 번호 + 데이터 파일 내에서 부여한 상대적 순번

로우 번호
- 블록 내 순번
```

### [인덱스를 스캔하는 이유]

검색 조건을 만족하는 소량의 데이터를 빨리 찾고 거기서 ROWID를 얻기 위해서 입니다. ROWID를 얻으면 테이블 레코드를 찾아갈 수 있습니다.

### [인덱스 탐색 과정]

인덱스 탐색 과정은 두 가지로 나눠집니다.

- 수직적 탐색 : 인덱스 스캔 시작 지점을 찾는 과정
- 수평적 탐색 : 데이터를 찾는 과정

### [인덱스 수직적 탐색]

수직적 탐색은 조건을 만족하는 레코드를 찾는 과정이 아닌 조건을 만족하는 첫 번째 레코드를 찾는 과정입니다. 즉, 조건을 만족하는 첫 번째 레코드가 목표입니다.

루트 블록에서부터 시작해 리프 블록까지 수직적 탐색을 진행합니다.

(이떄, 루트 블록, 브랜치 블록에는 각 하위 블록에 대한 주소값을 가지고 있습니다.)

### [인덱스 수평적 탐색]

수평적 탐색은 인덱스에서 본격적으로 데이터를 찾는 과정으로 수직적 탐색을 통해 스캔 시작점을 찾고, 찾고자 하는 데이터가 더 안 나타날때까지 인덱스를 스캔합니다.

인덱스 리프 블록 탐색을 수평적으로 진행합니다.

(이때, 인덱스 리프 블록은 양 방향 연결 리스트로 연결되어 앞뒤 서로 주소값을 가지고 있습니다.)

### [수직적 탐색을 한 후 수평적 탐색을 하는 이유]

인덱스를 수평적 탐색하는 이유는

- 조건절을 만족하는 데이터를 모두 찾기 위해서입니다.
- ROWID를 얻기 위해서입니다.
(필요한 컬럼을 인덱스가 모두 갖고 있어 인덱스만 스캔해도 되지만, 일반적으로 인덱스를 스캔하고서 테이블도 액세스하기 때문에 ROWID가 필요합니다.

### [결합 인덱스 구조와 탐색]

```
결합 인덱스
- 두 개 이상 컬럼을 결합해 생성한 인덱스로 리프 블록은 인덱스 키값 순으로 정렬됩니다.
```

수직적 탐색을 거쳐 찾은 인덱스 스캔 스작점에서 인덱스 선두 컬럼이 모두 동등조건으로 검색된다면 어느 컬럼을 인덱스 앞쪽에 두든 블록 I/O 개수가 같습니다.

즉, 인덱스 선두 컬럼이 모두 동등 조건이면 순서에 상관없이 성능이 같습니다.

B*Tree 인덱스는 엑셀처럼 편면 구조가 아닌 다단계 구조이기 떄문에 어느 컬럼을 앞에 두든 일량에는 차이가 없습니다.

## 인덱스 기본 사용법

인덱스 기본 사용법은 Range Scan 하는 방법입니다.

인덱스 확장기능은 Index Range Scan 이외의 다양한 스캔 방법을 말합니다.

### [인덱스 사용]

```
Index Full Scan
- 스캔 시작점을 찾을 수 없고 멈출 수 없어 리프 블록 전체를 스캔하는 방식
Index Range Scan
- 리프 블록에서 스캔 시작점을 찾아 거기서부터 스캔하다가 중간에 멈출 수 있는 리프 블록 일부를 스캔하는 방식
- 인덱스를 정상적으로 사용한다라는 표현을 사용
```

인덱스 컬럼을 가공하면 인덱스를 정상적으로 사용할 수 없습니다.

즉, 인덱스 스캔 시작점을 찾을 수 없고 인덱스에서 일정 범위를 스캔할 수 없습니다.

- 인덱스 컬럼을 LIKE 연산자로 중간값 검색
- 인덱스 컬럼에 OR 조건 사용
- 인덱스 컬럼에 IN 조건 사용

이러한 경우 인덱스 컬럼의 스캔 시작점을 알 수 없습니다.

```
인덱스를 정상적으로 사용한다.
- 리프 블록에서 스캔 시작점을 찾아 거기서부터 스캔하다가 중간에 멈추는 것
```

### [IN-List Iterator]

```
IN-List Iterator
- IN List 개수만큼 Index Range Scan을 반복해 SQL을 UNION ALL 방식으로 변환한 것과 같은 효과를 얻음
```

IN 조건은 OR 조건을 표현하는 다른 방식일 뿐이지만 SQL을 UNION ALL 방식으로 작성하면, 각 브랜치 별로 인덱스 스캔 시작점을 찾을 수 있어 

IN 조건절에는 SQL 옵티마이저가 IN-List Iterator 방식을 사용합니다.

### [인덱스 사용 조건]

```
Index Range Scan 조건
- 인덱스 선두 컬럼이 가공되지 않은 상태로 조건절에 있어야 함
```

즉, 인덱스 선두 컬럼이 가공되지 않고 조건절에 있다면 그 쿼리는 Index Range Scan을 진행합니다.

### [인덱스 튜닝에 관해서]

인덱스를 탄다 = 인덱스를 Range Scan 한다와 같은 의미입니다.

하지만, 인덱스를 정말 잘 타는지는 인덱스 리프 블록에서 스캔하는 양을 따져봐야 합니다.

### [인덱스를 이용한 소트 연산 생략]

테이블과 달리 인덱스는 정렬이 되어 있습니다. 정렬돼 있기 때문에 Range Scan이 가능하고, 소트 연산 생략 효과도 부수적으로 얻을 수 있습니다.

PK 인덱스를 이용해 동등 조건으로 검색하면 결과집합은 SQL에 ORDER BY가 있어도 정렬연산을 따로 수행하지 않습니다.

즉, 옵티마이저는 PK 인덱스를 출력한 결과집합은 어차피 정렬이 된 순이니 따로 정렬연산을 수행하지 않습니다.

인덱스 리프 블록은 양방향 연결 리스트 구조이기에 내림차순 정렬에도 인덱스를 사용할 수 있습니다.

### [조건절 이외의 컬럼 가공]

조건절에서 인덱스 컬럼을 가공하면 인덱스를 정상적으로 사용할 수 없습니다.

- ORDER BY
- SELECT-LIST

위 두가지에서 컬럼을 가공하면 인덱스를 정상적으로 사용할 수 없습니다.

1. ORDER BY
    
    ```sql
    SELECT *
    FROM 상태변경이력
    WHERE 장비번호 = 'C'
    ORDER BY 변경일자 || 변경순번
    ```
    
    인덱스에는 가공하지 않은 상태로 값을 저장했는데, 가공한 값을 기준으로 정렬하는 경우
    
2. SELECT-LIST
    
    ```sql
    SELECT NVL(MAX(TO_NUMBER(변경순번)),0)
    FROM 상태변경이력
    WHERE 장비번호 = 'C'
    AND 변경 일자 = '20180316'
    ```
    
    인덱스에는 문자열 기주능로 정렬돼 있는데, 이를 숫자값을 바꾼 값 기준으로 최종 변경 순번을 요구했기 때문에 정렬 연산이 들어갑니다.
    

### [자동 형변환]

각 조건절에서 양쪽 값의 데이터 타입이 서로 다르면 값을 비교할 수 없습니다.

이럴때 자동으로 형변환해주는 DBMS도 있기 때문에 주의해야 합니다.

- 숫자형 컬럼을 LIKE 조건으로 검색하면 자동 형변환이 발생합니다.
- DECODE 처리 시 자동 형변화이 발생합니다.

자동 형변환에 의존하지 말고, 인덱스 컬럼 기준으로 반대편 컬럼 또는 값을 정확히 형변환해주어야 합니다.

개발자가 형변환 함수를 생략하면 성능이 좋아질 것이라 생각하지만 옵티마이저는 자동으로 생성해 줍니다.

## 인덱스 확장기능 사용법

- Index Range Scan
- Index Full Scan
- Index Unique Scan
- Index Skip Scan
- Index Fast Full Scan

### [Index Range Scan]

```
Index Range Scan
- B*Tree 인덱스의 가장 일반적으로 정상적인 형태의 액세스 방식
- 인덱스 루트에서 리브 블록까지 수직적으로 탐색한 후 필요한 범위만 스캔
```

사용 조건 : 선두 컬럼을 가공하지 않은 상태로 조건절에 사용해야 합니다.

성능 : 인덱스 스캔 범위, 테이블 액세스 횟루를 얼마나 줄일 수 있느냐로 결정됩니다.

### [Index Full Scan]

```
Index Full Scan
- 수직적 탐색 없이 인덱스 리프 블록을 처음부터 끝까지 수평적으로 탐색하는 방식
```

데이터 검색을 위한 최적의 인덱스가 없을 때 차선으로 선택

- 인덱스 선두 컬럼이 조건절에 없다면 Table Full Scan을 고려하지만 부담이 크다면, Index Full Scan 방식을 선택
- 소트 연산을 생략함으로써 전체 집합 중 처음 일부를 빠르게 출력할 목적으로도 Index Full Scan 방식을 선택
    
    (Range Scan과 마찬가지로 결과집합이 인덱스 컬럼 순으로 정렬되기에 소트 연산이 생략가능합니다.)
    

주의할 점은 데이터를 끝까지 읽는 다면 Table Full Scan보다 훨씬 더 많은 I/O를 일으키고 결과적으로 수행 속도도 느려집니다.

### [Index Unique Scan]

```
Index Unique Scan
- 수직적 탐색만으로 데이터를 찾는 스캔 방식
```

사용 조건 : Unique 인덱스를 동등 조건으로 탐색하는 경우 작동합니다.

동작 방식 : Unique 인덱스가 존재하는 컬럼은 중복 값이 입력되지 않게 DBMS가 데이터 정합성을 관리해주기 때문에 해당 인덱스 키 컬럼을 모두 동등 조건으로 검색할 때는 데이터를 한 건 찾는 순간 탐색을 멈춥니다.

Unique 인덱스라도 Index Range Scan을 하는 경우도 존재합니다.

- 동등 조건이 아닌 범위 검색 조건
- 결합 인덱스에 대해 일부 컬럼만으로 검색 시

### [Index Skip Scan]

```
Index Skip Scan
- 인덱스 선두 컬럼이 조건절에 없어도 인덱스를 활용할 수 있는 방식
- 루트 또는 브랜치 블록에서 읽은 컬럼 값 정보를 이용해 조건절에 부합하는 레코드를 포함할 가능성이 있는 리프 블록만 골라서 액세스하는 스캔 방식
```

조건절에 빠진 인덱스 선두 컬럼의 Distainct Value 개수가 적고 후행 컬럼의 Dsitinct Value 개수가 많을 떄 효과적입니다.

작동하기 위한 조건이 있습니다.

- Distinct Value가 적은 두 개의 선두컬럼이 모두 조건절에 없는 경우
- 선두 컬럼이 범위 검색 조건일 때

### [Index Fast Full Scan]

```
Index Fast Full Scan
- 논리적인 인덱스 트리 구조를 무시하고 인덱스 세그먼트 전체를 Multiblock I/O 방식으로 스캔하는 방식
```

- Index Full Scan : 인덱스의 논리적 구조를 따라 블록을 읽어 드립니다.
- Index Fast Full Sacn : 물리적으로 디스크에 저장된 순서대로 인덱스 리프 블록들을 Multiblock I/O방식으로 읽어 드립니다.

장점 : 속도가 빠르고 디스크로부터 대량의 인덱스 블록을 읽어야 할 떄 큰 효과를 발휘합니다. 또한, 병렬스캔이 가능합니다.

단점 : 인덱스 리프 노드가 갖는 연결 리스트 구조 무시하기 때문에 정렬이 되어 있지 않고, 뭐리에 사용한 컬럼이 모두 인덱스에 포함되어야 합니다.

### [Index Range Scan Descending]

```
Index Range Scan Descending
- Index Ragne Scan과 기본적으로 동일한 스캔 방식
- 인덱스를 뒤에서부터 앞쪽으로 스캔하는 방식
```

내림차수능로 정렬된 결과집합을 얻을 수 있습니다.
