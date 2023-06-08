# User-defined Function

* SQL문 안에서 사용할 수 있는 함수를 사용자가 직접 정의

  - 이후 데이터 전처리 및 SparkML 사용시 자주 활용

<br/>

## 예시

* 데이터프레임

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/f8216e81-930d-4772-9e6d-70d62b1e3ba7)

<br/>

### 1. price를 제곱하는 함수 정의

  - 방법 1
  ```Python
  from pyspark.sql.types import LongType 
  
  def squared1(s):
    return s * s

  spark.udf.register("squared1", squared1, LongType())
  ```
    - 함수를 정의하고, register()를 사용해 등록
    - 결과값의 default type은 string이므로, register() 함수의 인자에 원하는 type으로 직접 지정   
      (LongType: 8-byte signed integer numbers)

  
  - 실행 결과

  ```Python
  spark.sql("SELECT name, squared1(price) from transactions").show()
  ```
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/efc8bec2-1848-48f2-b9d9-d10fbe38aaa4)


  - 방법 2
  ```Python
  from pyspark.sql.functions import udf
  
  @udf("long") 
  def squared2(s):
    return s * s

  spark.udf.register("squared2", squared2)
  ```
    - 함수를 정의하고, register()를 사용해 등록하는 것은 동일
    - 결과값의 type은 파이썬 데코레이터 형식으로 지정
    
  - 실행 결과 : 방법1과 동일한 결과

  ```Python
  spark.sql("SELECT name, squared1(price), squared2(price) from transactions").show()
  ```
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/900d63a8-724a-4cbd-9df9-9c9358b0fa7a)
  
  - 스키마도 확인 - 방법1,2 모두 결과값이 long type
  
  ```Python
  spark.sql("SELECT name, squared1(price), squared2(price) from transactions").printSchema() 
  ```
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/835fc97a-82fa-4d03-ba06-e4726339fcf6)
