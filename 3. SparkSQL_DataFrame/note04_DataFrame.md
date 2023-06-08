# DataFrame
- SparkSQL이 아닌 DataFrame?
- 사용법
    - 데이터 타입
    - 가능 연산
    - aggregation

## 관계형 데이터
- RDD는 (사용자가 정의하는 함수 사용) 함수형 데이터, DataFrame은 RDD+Relation인 관계형 데이터
- RDD는 함수형, DataFrame은 선언형
- 자동 최적화: 스키마가 있어서 자동 최적화
- 타입 없음: 타입을 강제하지 않음

## 특징
- RDD의 확장
- 지연 실행
- 분산 저장
- immutable(값이 변하지 않는다)
- row 객체 있음
- sql 쿼리 실행가능
- 스키마가 있어 최적화 가능
    ```python
    '''
    스키마 보는 법
    '''
    print(data.dtypes)
    # [('col1', 'string),]

    data.show() # 첫 20개 row 보여줌

    data.printSchema() # 트리 형태로 스키마 보여줌, 중첩된 스키마는 프린트스키마로 보는 것이 편함
    # root
    #   |-- col1: string (nullable=true)
    #   ...
    ```
- csv, json, hive를 읽거나 변환 가능

## Operation
- SQL과 비슷한 연산 가능
    - select: 사용자가 원하는 컬럼이나 데이터 추출
        ```python
        # 참고. collect()는 실행한다는 의미 

        df.select(*).collect()
        # [Row(age=2, name='Alice'), Row(age=5, name='Bob')]
        df.select('name','age').collect()
        # [Row(age=2, name='Alice'), Row(age=5, name='Bob')]
        df.select(df.name, (df.age+10).alias('age')).collect() # 연산 추가 & 컬럼명 지정
        # [Row(age=12, name='Alice'), Row(age=15, name='Bob')]
        ```
    - where
    - limit
    - orderby
    - groupby: 사용자 지정 컬럼을 기준으로 그룹핑
        ```python
        df. groupBy().age().collect()
        # [Row(avg(age)=3.5)]

        sorted(df.roupBy(df.name).avg().collect())
        # [Row(name='Alice', age=2.0), Row(name='Bob', age=5.0)]
        ```
    - join
        ```python
        df.join(df2, 'name').select(df.name, df2.height).collect()
        # [Row(name='Bob', height=85)]
        ```
        - 추가 예제
        ```python
        from pyspark.sql import SparkSession, Row
        from pyspark.sql import StructType, StructField, StringType, IntegerType
        # 데이터 선언 시 알아서 스키마 유추해 지정 
        # -> 아래 2984 값 있는 컬럼은 알아서 int로, 나머지는 스트링
        stocks = [('Google','GOOGL', 'USA', 2984, 'USD'), ...]

        schema = ["name","ticker","country","price","currency"]

        df = spark.createDataFrame(data = stocks, schema=schema)

        df.show()

        # 미국주식만 뽑아서 정렬
        usaStockDF = df.select("name", "country", "price").\
            where("country==USA").\
                orderBy("price")

        usaStockDF.show()

        # 통화 별 가격 최대값
        df.groupBy("currency").max("price").show()

        # !!! 함수 적용하기
        from pyspark.sql.functions import avg, count
        # 통화별 주가 평균
        df.groupBy("currency").agg(avg("price")).show()
        # 통화별 주식 수
        df.groupBy("currency").agg(count("price")).show()
        ```

# Aggregation
- 그룹핑 후 데이터를 하나로 합치기
    ```python
    df.agg({"age":"max"}).collect() # dictionary 컬럼:함수
    # [Row(max(age)=5)]

    from pyspark.sql import function as F # function에서 함수 불러와서 적용해도됨
    df.agg(F.min(df.age)).colect() 
    # [Row(min(age)=2)]
    ```