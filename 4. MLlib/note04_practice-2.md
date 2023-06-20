# 실습 - ALS 추천 알고리즘 파이프라인 구현

* Spark Session 생성
```Python

# Spark Session 생성
# 메모리 부족 문제 방지를 위해 직접 max memory 지정
from pyspark.sql import SparkSession

MAX_MEMORY = "5g" 
spark = SparkSession.builder.appName("movie-recommendation")\
    .config("spark.executor.memory", MAX_MEMORY)\ # spark executor의 메모리도 max설정값으로 
    .config("spark.driver.memory", MAX_MEMORY)\ # spark driver의 메모리도 max설정값으로
    .getOrCreate()
```

* 데이터 로드
```Python

# 영화 평점 데이터 로드
ratings_file = "/Users/asd31/jupyter_dongmin/data-engineering/01-spark/data/ml-25m/ratings.csv"
ratings_df = spark.read.csv(f"file:///{ratings_file}", inferSchema=True, header=True)

ratings_df.show()

'''
+------+-------+------+----------+
|userId|movieId|rating| timestamp|
+------+-------+------+----------+
|     1|    296|   5.0|1147880044|
|     1|    306|   3.5|1147868817|
|     1|    307|   5.0|1147868828|
|     1|    665|   5.0|1147878820|
|     1|    899|   3.5|1147868510|
|     1|   1088|   4.0|1147868495|
|     1|   1175|   3.5|1147868826|
|     1|   1217|   3.5|1147878326|
|     1|   1237|   5.0|1147868839|
|     1|   1250|   4.0|1147868414|
|     1|   1260|   3.5|1147877857|
|     1|   1653|   4.0|1147868097|
|     1|   2011|   2.5|1147868079|
|     1|   2012|   2.5|1147868068|
|     1|   2068|   2.5|1147869044|
|     1|   2161|   3.5|1147868609|
|     1|   2351|   4.5|1147877957|
|     1|   2573|   4.0|1147878923|
|     1|   2632|   5.0|1147878248|
|     1|   2692|   5.0|1147869100|
+------+-------+------+----------+
'''

# 필요컬럼만 남김
ratings_df = ratings_df.select(["userId", "movieId", "rating"])

# 스키마 확인
ratings_df.printSchema()

'''
root
 |-- userId: integer (nullable = true)
 |-- movieId: integer (nullable = true)
 |-- rating: double (nullable = true)
'''

```

* 기초통계량 확인
```Python

# 평점의 기초통계량 확인: 최소 0.5점 / 최대 5점
ratings_df.select("rating").describe().show()

'''
+-------+------------------+
|summary|            rating|
+-------+------------------+
|  count|          25000095|
|   mean| 3.533854451353085|
| stddev|1.0607439611423508|
|    min|               0.5|
|    max|               5.0|
+-------+------------------+
'''

```

* ALS 모델 학습
```Python

# train / test data splt
train_df, test_df = ratings_df.randomSplit([0.8, 0.2])

# ALS 모델 생성
from pyspark.ml.recommendation import ALS

als = ALS(
    maxIter=5,                   # 최고 반복 횟수
    regParam=0.1,                # 정규화
    userCol="userId",            # 유저 컬럼 지정
    itemCol="movieId",           # 아이템 컬럼 지정
    ratingCol="rating",          # 평점 컬럼 지정
    coldStartStrategy="drop"     # 콜드스타트 데이터 처리 -> nan/drop 중 선택
)

# 학습
model = als.fit(train_df)

# test set으로 예측
predictions = model.transform(test_df)
predictions.show()

'''
+------+-------+------+----------+
|userId|movieId|rating|prediction|
+------+-------+------+----------+
|   155|   1580|   2.5| 3.4590492|
|   321|   1580|   3.0|  3.111039|
|   368|   1580|   3.5|  3.726962|
|   385|   1088|   3.0|  3.001448|
|   472|   3918|   3.0| 2.4164915|
|   481|   1580|   4.0|   3.70944|
|   497|   1580|   5.0| 3.0903766|
|   513|  44022|   5.0| 4.3409266|
|   516|    833|   3.0|  2.800972|
|   587|   1580|   5.0| 3.9562593|
|   587|   6466|   4.0|  3.488453|
|   588|   1580|   2.5| 2.6375322|
|   597|   1580|   4.0|  3.666801|
|   597|   1591|   2.0| 2.5245519|
|   597|   1645|   5.0| 3.3962035|
|   606|   1580|   5.0| 4.2249255|
|   606|  36525|   2.5| 4.2196107|
|   606| 160563|   4.0|   4.01862|
|   626|   1088|   4.0| 3.3633018|
|   633|   1591|   5.0|  3.518829|
+------+-------+------+----------+
'''

# 실제 평점과 예측 평점의 통계량 비교 
## 평균은 비슷, 다만 예측평점은 -1.14~6.56으로 정상범위를 넘는 경우도 있음
predictions.select('rating', 'prediction').describe().show()

'''
+-------+------------------+------------------+
|summary|            rating|        prediction|
+-------+------------------+------------------+
|  count|           4995221|           4995221|
|   mean|3.5336134277142093|3.4213977694065445|
| stddev|1.0605956989607985|0.6434147966154989|
|    min|               0.5|        -1.1411085|
|    max|               5.0|         6.5618773|
+-------+------------------+------------------+
'''

```

* 모델 평가
```Python

from pyspark.ml.evaluation import RegressionEvaluator

# evaluator 컴포넌트 생성
evaluator = RegressionEvaluator(metricName='rmse', labelCol='rating', predictionCol='prediction')

# 예측결과를 넣어 평가
rmse = evaluator.evaluate(predictions)

# RMSE (성능이 좋지는 않지만 파이프라인 구현에 의의)
print(rmse)

'''
0.8088780174465089
'''

```

* 실제 추천결과 출력
```Python

# ALS에 내장된 함수 사용

# 1. 유저별 top3 영화 추천
model.recommendForAllUsers(3).show()

'''
+------+--------------------+
|userId|     recommendations|
+------+--------------------+
|    28|[{177209, 7.46965...|
|    31|[{177209, 3.99431...|
|    34|[{177209, 6.06151...|
|    53|[{194334, 6.95767...|
|    65|[{149484, 6.86115...|
|    78|[{177209, 7.05879...|
|    81|[{200930, 5.32819...|
|    85|[{149484, 5.79808...|
|   101|[{203086, 5.36891...|
|   108|[{149484, 5.69486...|
|   115|[{177209, 6.40233...|
|   126|[{203086, 6.58937...|
|   133|[{203086, 5.52253...|
|   137|[{203086, 5.57281...|
|   148|[{177209, 5.89362...|
|   155|[{202231, 6.02274...|
|   183|[{177209, 6.10260...|
|   193|[{177209, 5.39314...|
|   210|[{139036, 8.41384...|
|   211|[{203086, 6.66992...|
+------+--------------------+
'''

# 2. 영화별 top3 선호 유저 추천
model.recommendForAllItems(3).show()

'''
+-------+--------------------+
|movieId|     recommendations|
+-------+--------------------+
|     12|[{87426, 5.14755}...|
|     26|[{18230, 5.063113...|
|     27|[{87426, 5.328868...|
|     28|[{58248, 5.644647...|
|     31|[{87426, 5.060393...|
|     34|[{32202, 5.441312...|
|     44|[{87426, 5.228107...|
|     53|[{58248, 5.241786...|
|     65|[{30067, 5.231204...|
|     76|[{87426, 5.168890...|
|     78|[{142811, 4.75086...|
|     81|[{156318, 4.82232...|
|     85|[{137978, 4.87742...|
|    101|[{142811, 5.07395...|
|    103|[{96471, 5.063787...|
|    108|[{149507, 5.50686...|
|    115|[{105801, 5.77626...|
|    126|[{87426, 4.567394...|
|    133|[{108346, 5.04601...|
|    137|[{26659, 4.996}, ...|
+-------+--------------------+
'''

```

* 서비스 적용
    - 실제 서비스에서는 API 형태로 각 유저를 위한 추천을 제공해야 함
```Python
from pyspark.sql.types import IntegerType

# 3명의 유저에게 추천한다고 가정
user_list = [65, 78, 81]
users_df = spark.createDataFrame(user_list, IntegerType()).toDF('userId')

users_df.show()
'''
+------+
|userId|
+------+
|    65|
|    78|
|    81|
+------+
'''

# 3명의 유저에게 영화 5개씩 추천
## ALS 내장함수 사용 
user_recs = model.recommendForUserSubset(users_df, 5)
user_recs.show()

'''
+------+--------------------+
|userId|     recommendations|
+------+--------------------+
|    65|[{196717, 6.43577...|
|    78|[{203086, 6.67400...|
|    81|[{203086, 4.67855...|
+------+--------------------+
'''

# 실제 결과데이터는 이렇게 생김
user_recs.collect()

'''
[Row(userId=65, recommendations=[Row(movieId=196717, rating=6.435771465301514), Row(movieId=194434, rating=6.192673206329346), Row(movieId=157791, rating=6.140571594238281), Row(movieId=198535, rating=6.132567405700684), Row(movieId=144202, rating=6.010857582092285)]),
 Row(userId=78, recommendations=[Row(movieId=203086, rating=6.674009323120117), Row(movieId=194434, rating=6.530107498168945), Row(movieId=174627, rating=6.221368312835693), Row(movieId=203882, rating=6.195621013641357), Row(movieId=196787, rating=6.104584693908691)]),
 Row(userId=81, recommendations=[Row(movieId=203086, rating=4.678553104400635), Row(movieId=193257, rating=4.428552627563477), Row(movieId=116847, rating=4.337314605712891), Row(movieId=138580, rating=4.312744140625), Row(movieId=151989, rating=4.296421527862549)])]
'''

# 첫번째 유저의 실제 추천 영화리스트 추출
movies_list = user_recs.collect()[0].recommendations 
recs_df = spark.createDataFrame(movies_list)
recs_df.show()

'''
+-------+-----------------+
|movieId|           rating|
+-------+-----------------+
| 196717|6.435771465301514|
| 194434|6.192673206329346|
| 157791|6.140571594238281|
| 198535|6.132567405700684|
| 144202|6.010857582092285|
+-------+-----------------+
'''

# movieId가 아니라 실제 영화제목을 붙여서 제공

## 영화제목이 담긴 테이블을 로드
movies_file = "/Users/asd31/jupyter_dongmin/data-engineering/01-spark/data/ml-25m/movies.csv"
movies_df = spark.read.csv(f"file:///{movies_file}", inferSchema=True, header=True)
movies_df.show()

'''
+-------+--------------------+--------------------+
|movieId|               title|              genres|
+-------+--------------------+--------------------+
|      1|    Toy Story (1995)|Adventure|Animati...|
|      2|      Jumanji (1995)|Adventure|Childre...|
|      3|Grumpier Old Men ...|      Comedy|Romance|
|      4|Waiting to Exhale...|Comedy|Drama|Romance|
|      5|Father of the Bri...|              Comedy|
|      6|         Heat (1995)|Action|Crime|Thri...|
|      7|      Sabrina (1995)|      Comedy|Romance|
|      8| Tom and Huck (1995)|  Adventure|Children|
|      9| Sudden Death (1995)|              Action|
|     10|    GoldenEye (1995)|Action|Adventure|...|
|     11|American Presiden...|Comedy|Drama|Romance|
|     12|Dracula: Dead and...|       Comedy|Horror|
|     13|        Balto (1995)|Adventure|Animati...|
|     14|        Nixon (1995)|               Drama|
|     15|Cutthroat Island ...|Action|Adventure|...|
|     16|       Casino (1995)|         Crime|Drama|
|     17|Sense and Sensibi...|       Drama|Romance|
|     18|   Four Rooms (1995)|              Comedy|
|     19|Ace Ventura: When...|              Comedy|
|     20|  Money Train (1995)|Action|Comedy|Cri...|
+-------+--------------------+--------------------+
'''

## 영화제목 정보를 추천결과에 Join해 제공
recs_df.createOrReplaceTempView("recommendations")
movies_df.createOrReplaceTempView("movies")

query = """
SELECT *
FROM
    movies JOIN recommendations
    ON movies.movieId = recommendations.movieId
ORDER BY
    rating desc
"""
recommended_movies = spark.sql(query)
recommended_movies.show()

'''
+-------+--------------------+--------------------+-------+-----------------+
|movieId|               title|              genres|movieId|           rating|
+-------+--------------------+--------------------+-------+-----------------+
| 196717|Bernard and the G...|Comedy|Drama|Fantasy| 196717|6.435771465301514|
| 194434|   Adrenaline (1990)|  (no genres listed)| 194434|6.192673206329346|
| 157791|.hack Liminality ...|  (no genres listed)| 157791|6.140571594238281|
| 198535|Trick: The Movie ...|Comedy|Crime|Mystery| 198535|6.132567405700684|
| 144202|Catch That Girl (...|     Action|Children| 144202|6.010857582092285|
+-------+--------------------+--------------------+-------+-----------------+
'''

```

* 함수화
    - 실제 서비스시에는 하나의 함수로 사용하는게 좋음
```Python

def get_recommendations(user_id, num_recs):
    users_df = spark.createDataFrame([user_id], IntegerType()).toDF('userId')
    user_recs_df = model.recommendForUserSubset(users_df, num_recs)
    
    recs_list = user_recs_df.collect()[0].recommendations
    recs_df = spark.createDataFrame(recs_list)
    recommended_movies = spark.sql(query)
    return recommended_movies

# 456번 유저에게 10개의 영화를 추천해줘.
recs = get_recommendations(456, 10)

# 결과 출력
recs.toPandas()
```
![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/a9216eda-898b-4f5c-8f95-8663d980301b)

<br/>

* 사용 끝나고 꼭 session stop하기
```Python

spark.stop()

```
