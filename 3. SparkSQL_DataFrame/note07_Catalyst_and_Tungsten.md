# Catalyst Optimizer & Tungsten Project 작동원리

<br/>

## 1. Catalyst & Tungsten 이란

* Spark Backend는 두가지 엔진(Catalyst와 Tungsten)을 통해 최적화됨.

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/12f1f23f-78ec-47cb-93a3-c78cb11f64a9)

  - Catalyst: 사용자가 작성한 코드를 실행 가능한 최적화된 계획으로 바꾸는 엔진
  - Tungsten: 최적화된 계획을 받아 low-level에서 하드웨어 최대 성능을 낼 수 있도록 CPU/메모리 효율을 최적화하는 엔진

<br/>

## 2. Catalyst Optimizer

* 그 전에, Logical/Physical Plan 이란?

  ||<center>Logical Plan</center>|<center>Physical Plan</center>|
  |:---|:---|:---|
  |설명|- 수행할 모든 Transformation 단계에 대한 추상화 <br/> - 데이터가 어떻게 변환되어야 하는지 정의 <br/> - 그러나 실제 어디서, 어떻게 동작하는지는 정의하지 않음  | - Logical Plan이 어떻게 클러스터 위에서 실제 수행될지 정의 <br/> - 실행 전략을 만들고, Cost Model에 따라 최적화 |

<br/>

* Catalyst는 **Logical Plan을 Physical Plan으로 바꾸는 일을 수행**

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/122e5039-4aae-4553-9607-c9db68b819cc)

  1) 분석
      - DataFrame 객체의 relation을 계산, 컬럼의 타입과 이름을 확인
      - 사용자가 컬럼명을 잘못 설정하면 이 단계에서 error가 발생

  2) Logical Plan 최적화
      - 상수로 표현된 표현식을 Compile Time에 계산 (x runtime)
      - Predicate Pushdown: join & filter --> filter & join 으로 변경
      - Projection Pruning: 결과 연산에 필요한 컬럼만 가져오도록 함

  3) Physical Plan 생성
      - 최적화된 Logical Plan을 Spark에서 실제 실행 가능한 Plan으로 변환

  4) Code Generation
      - 최적화된 Physical Plan을 실제 CPU에서 돌릴 수 있도록 Java Bytecode로 변환

<br/>

* Catalyst의 최적화 예시

  ```SQL
  SELECT zone_data.Zone, count(*) AS trips
  FROM trip_data
  JOIN zone_data
    ON trip_data.PULocationID = zone_data.LocationID
  WHERE trip_data.hvfhs_license_num = 'HV0003'
  GROUP BY zone_data.Zone
  ORDER BY trips desc
  ```
  
  - Logical Planning 최적화는 아래 그림과 같이 이루어짐
    ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/5c0c0a61-ab3f-44dc-96ab-e6753acfb5c8)
      - (왼쪽 최적화 전) 아래부터 위로 Scan -> Aggregate까지, 사용자가 작성한 쿼리 그대로 Plan 생성
      - (오른쪽 최적화 후) WHERE 조건은 trip_data 테이블에만 적용되므로, join & filter 하는게 아니라 filter & join 하도록 Plan이 변경됨

<br/>

* 실제 코딩시: explain() 함수 사용
  - Logical Plan => Physical Plan 변환 과정을 출력


  ```Python
  spark.sql("SELECT zone_data.Zone, count(*) AS trips \
  FROM trip_data JOIN zone_data ON trip_data.PULocationID = zone_data.LocationID \
  WHERE trip_data.hvfhs_license_num = 'HV0003' \
  GROUP BY zone_data.Zone ORDER BY trips desc").explain(True)
  ```
  
  - 출력 결과
    1) Parsed Logical Plan: 사용자가 작성한 순서 그대로의 Plan   
      ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/8f0dc8ad-d5e9-4df0-966c-a9c68c55b53c)

    2) Analysed Logical Plan: 사용자가 지정한 테이블의 relation 및 컬럼 확인
      ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/baf9ae53-eb6b-449e-883f-9516d6e70cac)

    3) Optimized Logical Plan: filter / join 순서 등 최적화
      ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/46ced6a2-4fab-4710-91de-552c16c28fd4)

    4) Physical Plan: 실제 연산이 수행되기 위한 Plan
      ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/49af6d90-80db-4fdd-aabc-5905d7d3c56e)

<br/>

## 3. Tungsten project

* Catalyst Optimizer의 결과인 Physical Plan을 실행하기 위한 Code Generation 작업을 수행

  - Code Generation: 분산환경에서 실제 수행될 Bytecode를 만드는 작업
  - Cache를 활용한 연산 및 메모리 관리 최적화 => 성능 향상이 목적
