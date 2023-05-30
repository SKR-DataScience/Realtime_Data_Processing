# Key-Value RDD

## 1. Key-Value RDD란?

* Key-Value RDD: Key와 Value 쌍으로 된 데이터를 다루는 RDD

  - (Key, Value) 쌍을 갖기 때문에 Pairs RDD 라고도 불림
  - 예시   
  
    - 지역ID별 택시 운행 수는 어떻게 될까?
    
      - Key: 지역ID
      - Value: 택시 운행
      
        ~~~python
        [
          (지역ID, 운행 수)
          (지역ID, 운행 수)
        ]
        ~~~
      
    - 드라마 장르별 별점 평균 구하기, e-commerce 사이트에서 상품별 평균 리뷰 수 구하기 등

<br/>

* Key-Value RDD를 만드는 코드

  - 코드

    ~~~python
    pairs = rdd.map(lambda x: (x, [1,1]))
    ~~~

  - 결과

    ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/0f96c189-2f47-4b32-9cd8-b3b00cb41620)   
    (단순 값 뿐 아니라 리스트도 Value가 될 수 있음)
    
    
<br/>

## 2. Key-Value RDD로 할 수 있는 것들

1) Reduction
  ```
  - reduceByKey() : 키 값을 기준으로 태스크 처리
  - groupByKey() : 키 값을 기준으로 밸류를 묶음
  - sortByKey() : 키 값을 기준으로 정렬
  - keys() : 키 값 추출
  - values() : 밸류 값 추출
  ```
  
2) Join
  ```
  - join
  - rightOuterJoin
  - leftOuterJoin
  - subtractByKey
  ```
  
3) Mapping values
  ```
  - mapValues()
  - flatMapValues()
  
  - 특히 Key-Value 데이터에서 Key를 바꾸지 않는 경우
    - map() 대신, value만 다루는 mapValues() 함수를 써주자
    - RDD에서 key는 유지하고 Value만 연산으로 다룸
    - Spark 내부에서 파티션을 유지할 수 있어 더욱 효율적
  ```
  
