# Apache Spark

## 1. Apache Spark란

* 빅데이터 처리를 위한 오픈소스 고속 분산처리 엔진

  - Apache Spark는 이미 수많은 기업에서 사용중 (Amazon, MS, Uber, Netflix, Airbnb,...)
  - 이 기업들이 공통적인 문제를 가지고 있었다는 의미
    - 규모 : 데이터의 크기 증가
    - 속도 : 데이터가 생성되는 속도가 증가
    - 다양성 : 데이터의 종류 증가

* 스파크의 장점 : 빠르다.

  - 처리해야 할 데이터가 많을 때 -> '데이터를 쪼개서 처리하자'는 아이디어
  - 쪼갠 데이터를 여러 노드의 메모리에서 동시에 처리 (In-Memory 연산)


## 2. Spark의 구조

* 스파크는 하나의 Cluster를 이루게 됨

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/5af472c9-c18c-43c6-a459-c55499ea625a)

  1. Driver Program
      - 우리가 사용하게 될 CPU
      - Python, Java, Scala 등으로 Task를 정의, 생산

  2. Cluster Manager
      - 정의된 Task를 분배
      - 여러가지 종류가 있음 (ex. Hadoop - Yarm / AWS - Elastic MapReduce)

  3. Worker Node
      - 실제 In-Memory 연산을 수행
      - 이상적으론 1CPU 코어 당 1Node 배치

## 3. Pandas vs Spark

* 내 컴퓨터에서 당장 Spark를 돌렸을 때 Pandas보다 느린 이유

  - Spark는 확장성을 고려해 설계되었기 때문
  
  - 1개의 노드에서 Pandas / Spark 성능 비교
    : 배열에서 최고값을 찾는 단순 연산 수행
    
    ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/3d307030-cc9a-4ddd-8e4e-df240033b484)
    - Pandas는 작은 파일크기에서 속도가 빠르지만, 일정 파일크기 넘어가면 out of memory
    - Spark는 작은 파일크기에서 속도가 비교적 느리지만, 파일 크기가 증가해도 속도를 유지하며 연산 가능  
      (사실, 그래도 Hadoop 등 다른 프레임워크 보다는 빠름)
    - '수평적 확장'이 가능하기 때문: 노드를 필요에 따라 계속 늘릴 수 있음

<br/>

* Pandas / Spark 비교

|**Pandas**|**Spark**|
|:---|:---|
| 1개의 노드 | 여러개의 노드 |
| Eager Execution - 코드가 바로바로 실행 | Lazy Excution - 실행이 필요할 때까지 기다림 |
| 컴퓨터 하드웨어의 제한을 받음 | 수평적 확장이 가능 |
| In-Memory 연산 | In-Memory 연산 |
| Mutable Data | Immutable Data |
