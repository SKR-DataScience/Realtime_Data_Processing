# Transformations and Actions

<br/>

## 1. Transformation과 Action의 차이

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/ce94a8c6-ce28-4e6b-9dbb-5b1a980e3dbb)

### - Transformation
  - 새로운 RDD를 반환
  - **지연 실행 (Lazy Execution)**
  
### - Action
  - 결과값을 연산하여 출력하거나 저장
  - **즉시 실행 (Eager Execution)**

<br/>

#### 주요 함수 비교
- Transformation
  ```
  - map()
  - flatMap()
  - filter()
  - distnct()
  - reduceByKey()
  - groupByKey()
  - mapValues()
  - flatMapValues()
  - sortByKey()
  ```
  
- Actions
  ```
  - collect()
  - count()
  - countByValue()
  - take()
  - top()
  - reduce()
  - fold()
  - foreach()
  ```

- 각 함수 기능은 실습파일에 기록

<br/>

## 2. Transformation의 분류

||<center>Narrow Transformation</center>|<center>Wide Transformation</center>|
|:---|:---|:---|
|개념그림|<center>![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/3f97f6b7-83c3-492a-8381-83876a4e1ec0)</center>|<center>![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/b9ad4f50-555e-4488-86b8-9bdf579cbca5)</center>|
|특징|- 1:1 변환 <br/> - 하나의 열을 조작하기 위해 다른 파티션의 데이터 쓸 필요 없음 <br/> - 정렬이 필요하지 않은 경우 사용 | - Shuffling <br/> - 다른 파티션의 데이터가 섞여 변환 아웃풋 RDD에 들어가게 될 수 있음 |
|함수|- filter(), map(), flatMap(), sample(), union() | - Intersection, join, distinct, cartesian, reduceByKey(), groupByKey()    |

<br/>

## 3. 지연 실행(Lazy Execution)의 이점

* 데이터를 다루는 task는 iteration이 많으며, 이 과정에서 메모리를 최대한 효율적으로 쓰는 것이 중요 (ex. ML 학습)
  
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/fe042944-ea9a-452e-bf80-63876772b19c)
  
  - 하둡은 각 작업(step)이 끝날 때마다 disk에 저장. 비효율적.
  - 반면 Spark는 작업 후 결과를 메모리에 저장 후 다시 활용함. (In-memory)
    - Spark에서 지연 실행이 가능하기 때문에, 중간중간 메모리를 효과적으로 사용하도록 설계함으로써 더욱 효과적으로 최적화가 가능함


* Transformation 후 결과를 메모리에 저장하기 위한 함수: **Persist() / Cache()**
  
  - 예시
  ```python
  val lastYearsLogs: RDD[String]=...
  val logsWithError = lastYearsLogs.filter(_.contains("ERROR")).persist()
  val firstLogsWithErrors = logsWithErrors.take(10)
  val numErrors = logsWithErrors.count() //faster
  ```
  - String 데이터가 들어있는 RDD에서, 'ERROR'라는 글자가 들어간 것만 필터링 (Transformation)
  - 필터링한 결과를 persist() 함수를 통해 메모리에 저장
  - 메모리에 결과가 남아있기 때문에, 이후 take()나 count() 등 Action이 더 빨리 수행됨

  
  
- Persist와 Cache의 차이
  - Persist(): 파라미터를 통해 storage level을 지정 가능 (memory-only, disk-only, memory-and-disk,...)
  - Cache(): storage level이 항상 memory-only
