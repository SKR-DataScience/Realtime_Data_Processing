# 실습 1-1. 간단한 ML model 구현 (Logistic Regression) 

```Python

# 세션/인스턴스 생성
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local").appName("logistic-regression").getOrCreate()


# 필요한 패키지 불러오기
from pyspark.ml.linalg import Vectors
from pyspark.ml.classification import LogisticRegression


# DataFrame 생성
training = spark.createDataFrame([
    (1.0, Vectors.dense([0.0, 1.1, 0.1])),
    (0.0, Vectors.dense([2.0, 1.0, -1.0])),
    (0.0, Vectors.dense([2.0, 1.3, 1.0])),
    (1.0, Vectors.dense([0.0, 1.2, -0.5]))], ["label", "features"])


training.show()

'''
+-----+--------------+
|label|      features|
+-----+--------------+
|  1.0| [0.0,1.1,0.1]|
|  0.0|[2.0,1.0,-1.0]|
|  0.0| [2.0,1.3,1.0]|
|  1.0|[0.0,1.2,-0.5]|
+-----+--------------+
'''


# Logistic Regression 인스턴스 생성
lr = LogisticRegression(maxIter=30, regParam=0.01)


# 모델 생성 및 학습
model = lr.fit(training)


# 테스트 데이터 생성
test = spark.createDataFrame([
    (1.0, Vectors.dense([-1.0, 1.5, 1.3])),
    (0.0, Vectors.dense([3.0, 2.0, -0.1])),
    (1.0, Vectors.dense([0.0, 2.2, -1.5]))], ["label", "features"])


# 예측
prediction = model.transform(test)


# 예측 결과 출력
prediction.show()

'''
+-----+--------------+--------------------+--------------------+----------+
|label|      features|       rawPrediction|         probability|prediction|
+-----+--------------+--------------------+--------------------+----------+
|  1.0|[-1.0,1.5,1.3]|[-6.2435550918400...|[0.00193916823498...|       1.0|
|  0.0|[3.0,2.0,-0.1]|[5.45228608726759...|[0.99573180142693...|       0.0|
|  1.0|[0.0,2.2,-1.5]|[-4.4104172202339...|[0.01200425500655...|       1.0|
+-----+--------------+--------------------+--------------------+----------+
'''

```

# 실습 1-2. 간단한 ML Pipeline 생성 

```Python

# 세션/인스턴스 생성
from pyspark.sql import SparkSession
spark = SparkSession.builder.master("local").appName("logistic-regression").getOrCreate()

# 필요한 패키지 불러오기
from pyspark.ml import Pipeline
from pyspark.ml.linalg import Vectors
from pyspark.ml.classification import LogisticRegression
from pyspark.ml.feature import HashingTF, Tokenizer


# DataFrame 생성 (text data)
training = spark.createDataFrame([
    (0, "a b c d e spark", 1.0),
    (1, "b d", 0.0),
    (2, "spark f g h", 1.0),
    (3, "hadoop mapreduce", 0.0)
], ["id", "text", "label"])


# 글자를 스플릿하기 위한 Tokenizer와, 빈도수를 해쉬값으로 변환하는 해싱 TF 생성
tokenizer = Tokenizer(inputCol="text", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")


# Logistic Regression 인스턴스 생성
lr = LogisticRegression(maxIter=30, regParam=0.001)


# Pipeline 생성 (각 stage로 구성)
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])


# Pipeline에 따라 모델 생성 및 학습
model = pipeline.fit(training)


# 테스트 DataFrame 생성
test = spark.createDataFrame([
    (4, "spark i j k"),
    (5, "l m n"),
    (6, "spark hadoop spark"),
    (7, "apache hadoop")
], ["id", "text"])


# 예측
prediction = model.transform(test)


# 예측 결과 출력
prediction.select(["id", "text", "probability", "prediction"]).show()

'''
+---+------------------+--------------------+----------+
| id|              text|         probability|prediction|
+---+------------------+--------------------+----------+
|  4|       spark i j k|[0.63102699631690...|       0.0|
|  5|             l m n|[0.98489377609773...|       0.0|
|  6|spark hadoop spark|[0.13563147748816...|       1.0|
|  7|     apache hadoop|[0.99563405823116...|       0.0|
+---+------------------+--------------------+----------+
'''

```
