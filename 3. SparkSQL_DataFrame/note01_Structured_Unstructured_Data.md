# 데이터 종류 별 차이점

- 데이터의 종류에 따라 처리 방법이 다름
    1. Structured Data
    2. Unstructured Data
    3. Semistructured Data
- Structured Data는 SparkSQL을 통해 자동으로 연산 순서 최적화 가능
-----

##  연산순서
- 예제. 미국의 $2000 이상 주식만 가져오기 
    - 데이터를 합치고 추출하기
- ticker와 price를 inner join 후 국가 필터링 후 가격 필터링(case1) 또는 필터링과 조인 순서 반대로(case2)
    - inner join
    - filter by country
    - filter by currency

```python
from pyspark import SparkConf, SparkContext
conf = SparkConf().setMaster("local").setAppName("example")
sc = SparkContext(conf=conf)

tickers = sc.parallelize([
        (1, ("Google" , "GOOGL", "USA" )), # 회사이름, 주식 상 이름, 나라 이름
        (2, ("Netflix", "NFLX", "USA")),
        (3, ("Amazon" , "AMZN", "USA")) ,
        (4, ("Tesla", "TSLA", , "USA" )),
        (5, ("Samsung", "905930", "Korea")),
        (6, ("Kakao", "035720", "Korea"))
]）
prices = sc.parallelize([
        (1, (2984, "USD" )), # 가격, 화폐기준
        (2, (645, "USD")),
        (3, (3518, "USD")), 
        (4, (1222, "USD")),
        (5, (70600,"KRW")), 
        (6, (125000, "KRW"))
])
```

- join과 filter를 통해 $2000 이상인 미국 주식 가져오기
     - 방법은 여러가지이나 case2가 1보다 성능 좋다
     - case2는 먼저 필터링 했기 때문에 이후 조인 시 셔플링 수 줄어들어 연산이 더 적다.

```python
# CASE 1: join 먼저, f11ter 나중에
# ticker와 price를 inner join 후 국가 필터링 후 가격 필터링
tickerPrice = tickers. join(prices)
tickerPrice.filter(lambda x: x [1][0][2] == "USA" and x[1][1][0] > 2000).collect()
# [1, ('Google', "GOOGL", 'USA'), (2984, 'USD'))), (3, (('Amazon','AMZN','USA'), (3518, 'USD')))]

# CASE 2: filter 먼저, join 나중에
filteredTicker = tickers. filter(lambda x: x[1][2] == "USA")
filteredPrice = prices.filter(lambda x:[1][0] > 2000)
filteredTicker.join(filteredPrice).collect ()
# [1, (('Google', "GOOGL','USA'), (2984, 'USD'))), (3, (('Amazon','USA'), (3518,'USD')))]
```

## 데이터 포맷의 종류
1. Unstructured Data: 자유 포맷
    - 로그 파일
    - 이미지
2. Structured Data: 행, 열이 있음
    - CSV
    - JSON
    - XML
3. Semistructured Data: 행, 열 + 데이터타입(스키마)
    - 데이터베이스

## 구조화된 데이터(Structured Data)

- 효율적인 연산 순서를 개개인이 판단해야 한다면 사람마다 성능차이가 심함. 스파크가 알아서 가능?
- RDD에서는 데이터 구조를 모르기 때문에 개발자에게 데이터 핸들링 의존
  - map, flatMap, filter 등 유저가 만든 함수 수행
- Structured data에서는 데이터의 구조를 알고 있으므로 수행할 태스크만 정의하면 됨
  - (스키마를 통해) 자동으로 최적화 가능
  - <span style="color:orange;"> Spark SQL은 structured data에 대해서는 자동으로 연산 최적화! 유저가 함수 정의 안해도 작업 수행 가능 </span>