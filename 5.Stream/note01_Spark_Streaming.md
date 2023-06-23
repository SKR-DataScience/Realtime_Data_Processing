# Spark Streaming

* Spark SQL 위에 만들어진 분산 스트림 처리 프로세싱
* 데이터 스트림을 처리할 때 사용
* 시간대별로 데이터를 합쳐(aggregate) 분석할 수 있음
* 체크포인트를 만들어서, 부분적 결함이 발생하더라도 다시 돌아가 데이터를 처리할 수 있음
* Kafka, Amazon Kinesis, HDFS 등 다른 서비스와 연결 가능
<br/>

### 스트림 데이터 (Streaming Data)

![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/639c5f2e-75f5-4c3c-8280-9edd1ee11fbb)

- 스트림 데이터는 하나의 무한한 테이블같은 개념
- 무한한 데이터를 처리할 수 있는 여러가지 아이디어가 있는데, 그 중 '여러개의 데이터로 잘 쪼개 처리를 한다'는 아이디어
  - 인풋 데이터 스트림(input data stream)이 무한하게 들어오면, Sprak Steraming의 리시버(receiver)가 받아서 여러개로 잘 쪼개줌 (micro-batching)
  - 마이크로 배칭된 데이터(batches of input data)를 Spark Engine으로 넘겨줌. 엔진은 이를 데이터프레임과 비슷하게 처리함
<br/>

### DStreams (Discretized Streams)

![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/b29f9d4d-cb52-45d4-9fe3-e08c752451c4)

- Spark Streaming의 기본적인 추상화
- 내부적으로는 RDD의 연속이고, RDD의 속성을 동일하게 가짐
  - 불변성, 탄력성, 분산 저장 등
  - 디스트림에 transformation을 적용하여 새로운 디스트림을 만들 수 있음
    - Map / FlatMap / Filter / ReduceByKey 등
- 위 그림과 같이, 시간대별 스냅샷을 찍어 RDD가 연속으로 정리되고 그것이 하나의 디스트림을 만들게 됨
- lines 디스트림에 transformation을 가하여 words 디스트림이 생성됨
  - 하나의 문장을 가진 RDD에 transformation의 하나인 flatMap을 적용하여 각각의 단어가 나오게 하는 것
<br/>

### Window Operations

![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/6beae1e3-eeb1-42cd-8979-1527c63e33ec)

- 현재시점의 데이터를 처리하기 위해 이전 데이터에 대한 정보가 필요할 때,
  윈도우 오퍼레이션으로 데이터를 묶어서 처리할 수 있음
<br/>

### State

- 이전 데이터에 대한 정보를 state로 주고 받을 수 있음   
  - ex) 카테고리(Group)별 총 합 집계   
        - 1번째 데이터 - 1번째 데이터 합   
        - 2번째 데이터 - 1,2번째 데이터 합   
        - 3번째 데이터 - 1,2,3번째 데이터 합 ...   
<br/>


### Streaming Query 

#### - Source
- 데이터를 어디서 읽어올지 명시
- 여러 데이터 소스를 사용해 join(), union()으로 합쳐 쓸 수 있음
```Python
spark.readStram.format('kafka')
    .option('kafka.bootstrap.server', ...)
    .option('subscribe', 'topic')
    .load()
```

#### - Transformation
- 읽어온 데이터에 대한 transformation 작업쿼리 작성
    - selectExpr은 SQL문을 그대로 사용하여 연산할 수 있는 transformation 함수
```Python
spark.readStream.format('kafka')
    .option('kafka.bootstrap.servers', ...)
    .option('subscribe', 'topic')
    .load()
    .selectExpr('cast (value as string) as json')
    .select(from_json('json', schema). As('data'))
```

#### - Processing Details
- 변환한 데이터를 write하기 위한 여러 옵션을 작성
  - micro batch 실행 간격, checkpoint 위치 설정 등
```Python
spark.readStream.format('kafka')
    .option('kafka.bootstrap.servers', ...)
    .option('subscribe', 'topic')
    .load()
    .selectExpr('cast (value as string) as json')
    .select(from_json('json', schema). As('data'))
    .writeStream.format('parquet')
    .trigger('1 minute') # micro-batch 실행 간격
    .option('checkpointLocation', '...')
    .start()
```
