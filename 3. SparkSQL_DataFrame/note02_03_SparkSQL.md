# I. Spark SQL

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


# II. SparkSQL 코드 실습
- 주요 구성요소
    - 데이터 프레임 생성하기
        - df = spark.createDataFrame(data=stocks, schema=stockSchema) 
    - 스키마 지정하기
        - spark.createDataFrame(data=stocks, schema=stockSchema)
    - 테이블 등록하기
        - df.createOrReplaceTempView("stocks") 

## 1. 한 개 테이블

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local").appName("learn-sql").getOrCreate()
stocks = [
    ('Google', 'GOOGL', 'USA', 2984, 'USD'), 
    ('Netflix', 'NFLX', 'USA', 645, 'USD'),
    ('Amazon', 'AMZN', 'USA', 3518, 'USD'),
    ('Tesla', 'TSLA', 'USA', 1222, 'USD'),
    ('Tencent', '0700', 'Hong Kong', 483, 'HKD'),
    ('Toyota', '7203', 'Japan', 2006, 'JPY'),
    ('Samsung', '005930', 'Korea', 70600, 'KRW'),
    ('Kakao', '035720', 'Korea', 125000, 'KRW'),
]
stockSchema = ["name", "ticker", "country", "price", "currency"] # 스파크 스키마(컬럼에 해당)
df = spark.createDataFrame(data=stocks, schema=stockSchema) # 자동 스키마 타입 지정

df.dtypes # 스키마 별  데이터 타입
# [('name', 'string'),
#  ('ticker', 'string'),
#  ('country', 'string'),
#  ('price', 'bigint'),
#  ('currency', 'string')]

df.show()
# +-------+------+---------+------+--------+
# |   name|ticker|  country| price|currency|
# +-------+------+---------+------+--------+
# | Google| GOOGL|      USA|  2984|     USD|
# |Netflix|  NFLX|      USA|   645|     USD|
# | Amazon|  AMZN|      USA|  3518|     USD|
# |  Tesla|  TSLA|      USA|  1222|     USD|
# |Tencent|  0700|Hong Kong|   483|     HKD|
# | Toyota|  7203|    Japan|  2006|     JPY|
# |Samsung|005930|    Korea| 70600|     KRW|
# |  Kakao|035720|    Korea|125000|     KRW|
# +-------+------+---------+------+--------+


# 중요!!!!
df.createOrReplaceTempView("stocks") # temporary view에 디비 테이블처럼 등록해야 sql 사용 가능

spark.sql("select name from stocks").show()
spark.sql("select name, price from stocks where country like 'U%' and name not like '%e%'").show() # 국가 이름이 U로 시작하고 e로 끝나지 않는 주식만 가져오기
spark.sql(
    f'''select name, price, currency from stocks 
        where currency = 'USD' and 
        price > (select price from stocks where name = 'Tesla');''').show() # 환이 USD이고 Tesla보다 주가가 높은 주식 선택
spark.sql(
    f'''select sum(price) as sum_p, mean(price) as mean_p, count(price) as cnt
        from stocks 
        where country = 'Korea';''').show()  # 한국 회사 주식 수
# +------+-------+---+
# | sum_p| mean_p|cnt|
# +------+-------+---+
# |195600|97800.0|  2|
# +------+-------+---+
```

## 2. 두 개 테이블
```python
# 중요!!!! 스키마 타입 직접 입력
from pyspark.sql.types import StringType, FloatType, StructType, StructField

earnings = [
    ('Google', 27.99, 'USD'), 
    ('Netflix', 2.56, 'USD'),
    ('Amazon', 6.12, 'USD'),
    ('Tesla', 1.86, 'USD'),
    ('Tencent', 11.01, 'HKD'),
    ('Toyota', 224.82, 'JPY'),
    ('Samsung', 1780., 'KRW'),
    ('Kakao', 705., 'KRW')
]

earningsSchema = StructType([
    StructField("name", StringType(), True), # 필드 정의
    StructField("eps", FloatType(), True), # eps: earings per shape
    StructField("currency", StringType(), True),
])
earningsDF = spark.createDataFrame(data=earnings, schema=earningsSchema) # 회사가 버는 돈

earningsDF.dtypes # 위에서 지정한 데이터 타입 잘 등록됨
# [('name', 'string'), ('eps', 'float'), ('currency', 'string')]

earningsDF.createOrReplaceTempView("earnings") # sql 사용을 위해 테이블 등록

earningsDF.select("*").show() # 모든 데이터 가져오기
# +-------+------+--------+
# |   name|   eps|currency|
# +-------+------+--------+
# | Google| 27.99|     USD|
# |Netflix|  2.56|     USD|
# | Amazon|  6.12|     USD|
# |  Tesla|  1.86|     USD|
# |Tencent| 11.01|     HKD|
# | Toyota|224.82|     JPY|
# |Samsung|1780.0|     KRW|
# |  Kakao| 705.0|     KRW|
# +-------+------+--------+

# join
spark.sql(
    f'''select * 
        from stocks 
        join earnings 
        on stocks.name = earnings.name''').show() 
```