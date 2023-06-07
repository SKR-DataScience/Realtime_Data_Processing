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

## 2. Transformation의 분류

||<center>Narrow Transformation</center>|<center>Wide Transformation</center>|
|:---|:---|:---|
|개념그림|<center>![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/3f97f6b7-83c3-492a-8381-83876a4e1ec0)</center>|<center>![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/b9ad4f50-555e-4488-86b8-9bdf579cbca5)</center>|
|특징|- 1:1 변환 <br/> - 하나의 열을 조작하기 위해 다른 파티션의 데이터 쓸 필요 없음 <br/> - 정렬이 필요하지 않은 경우 사용 | - Shuffling <br/> - 다른 파티션의 데이터가 섞여 변환 아웃풋 RDD에 들어가게 될 수 있음 |
|함수|- filter(), map(), flatMap(), sample(), union() | - Intersection and join, distinct, cartesian, reduceByKey(), groupByKey()    |

