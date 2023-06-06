# Spark SQL

- \# SparkSQL, #DataFrame, #Datasets
---

## 사용 목적
1. 스파크 프로그래밍 내부에서 관계형 처리
2. 스키마 정보를 이용해 자동 최적화
3. 외부 데이터셋을 쉽게 사용

## 개요
- 스파크 위에 구현된 패키지
- 주요 API
    - SQL
    - DataFrame
    - Datasets
- 백엔드 컴포넌트
    - Catalyst: 쿼리 최적화 엔진
    - Tungsten: 시리얼라이저(용량 최적화)

## SparkSession
-  Spark Core의 SparkContext가 SparkSQL의 SparkSession에 해당
    ```python
    spark = Sparksession.builder.appName("test-app").getOrCreate()
    ```
## DataFrame
- Spark Core의 RDD가 Spark SQL의 DataFrame에 해당
- DataFrame은 테이블 데이터셋에 해당
- RDD에 스키마가 적용된 개념(RDD에 적용된 function, partition도 적용됨)
- 장점
    - 다른 스파크 모듈(MLLib, Spark Streaming)과 사용 용이
    - 개발 쉬움
    - 자동 최적화
## DataFrame 만들기
1. 방법1. RDD에서 스키마 정의 후 변형
    - (방법1.1) 스키마를 자동으로 유추해 DataFrame 만들거나
    - (방법1.2) 스키마를 사용자가 정의
    ```python
    # RDD 만들기
    lines = sc.textFile("example. csv")
    data = lines.map(lambda x: x.split(",")) # 각 라인을 배열로 바꿈
    preprocessed = data.map(lambda x: Row(name=x[0], price=int(x[1]))) # 각 열을 입력값으로 받아 Row object 만듦

    # (방법1.1) Infer: preprocessed라는 RDD를 자동으로 스키마를 유추해서 DataFrame 만들기
    df = spark.createDataFrame(preprocessed) 

    # (방법1.2) Specify: 각 열의 스키마 지정
    schema = StructType(
        Structfield("name", StringType(), True), # Strucfied object
        Structfield ("price", StringType(), True)
    )
    spark.createDataframe(preprocessed, schema).show()
    ```
2. 방법2. 파일(CSV, JSON 등)에서 받기
    ```python
    from pyspark.sql import Sparksession
    spark = SparkSession.builder.appName("test-app").getOrCreate()
    #JSON
    dataframe = spark.read.json('dataset/nyt2.json')
    #TXT FILES#
    dataframe_txt - spark.read.text('text_data.txt')
    #CSV FILES#
    dataframe_sv = spark.read.csv('csv_data.csv')
    #PAROUET FILES#
    dataframe_parquet = spark.read.load('parquet_data.parquet')
    ```
## python에서 SparkSQL 사용하기
- DataFrame을 데이터베이스 테이블처럼 사용하려면 createOrReplaceTempView() 함수로 temporary view 만들기
    - SQL문을 사용해 쿼리 가능
        ```python
        data.createOrReplaceTempView("mobility_data") # 닉네임 지정, 테이블 명에 해당
        spark.sql("SELECT pickup_datetime FROM mobility_data LIMIT 5").show()
        ```
    - 함수를 이용해 쿼리 가능
        ```python
        df.select(df['name'], df['age']+1).show()
        df.filter(df['age']>21).show()
        df.grouBy("age").count().show()
        ```
- Spark Session으로 불러오는 데이터는 DataFrame
    ```python
    from pyspark.sql import Sparksession
    spark = SparkSession.builder.appName("test-app").getOrCreate()
    #JSON
    dataframe = spark.read.json('dataset/nyt2.json')
    ```
- DataFrame을 RDD로 변환해 사용할 수는 있으나 RDD를 덜 사용하는 것이 좋음
    - rdd = df.rdd.map(tuple)

## Datasets
    - Type이 있는 DataFrame
    - pyspark에서는 고려 사항 X (파이썬이 타입을 깐깐하게 고려하는 언어가 아니라서)