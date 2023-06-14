# MLlib 소개

## 1. MLlib이란?

![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/aad6d88e-ef47-42f3-99e5-5a1f7714b2f1)

* 'Machine Learning Library'
* **ML을 확장성 있게 적용**하고, **ML Pipeline을 쉽게 개발** 하기 위해 만들어진 스파크의 컴포넌트(Component) 중 하나
  - 스파크 위에는 여러 컴포넌트들이 있는데, 대부분 DataFrame API 위에서 작동   
    (RDD API는 지원이 끊기고 있는 추세임)

<br/>

## 2. MLlib의 기능

* ML Pipeline 구성 : **데이터 로딩 -> 전처리 -> 모델 학습 -> 모델 평가** (파라미터 튜닝 후 전체 과정 반복)
  - 여러 stage를 담고 있음
  - 저장될 수 있음 (persit)

* MLlib은 이러한 ML Pipeline을 만들기 위한 여러 기능들을 제공

![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/cb6be371-d2ae-4a9d-8cd5-ca8af0aab5fa)

  
* MLlib으로 할 수 있는 것들

  - Feature Engineering
  - 통계적 연산
  - ML Algorithms
    - Linear/Logistic Regression
    - Support Vector Machines
    - Naive Bayes
    - Decision Tree
    - K-Means Clustering
  - 추천 (Alternating Least Squares)


<br/>

## 3. MLlib의 주요 컴포넌트

### 1) Transformer

  - 피쳐 변환 및 학습된 모델의 추상화 기능
  - 데이터를 학습 가능한 포맷으로 변환
  - 모든 Transformer는 transform() 함수를 가지고 있음
    - DataFrame을 받아 새로운 DataFrame을 반환
      - 이 때, 보통 하나 이상의 column을 생성하게 된다
      - ex) Data Normalization, Tokenization, One-hot encoding

### 2) Estimator

  - 모델의 학습 과정 추상화 기능
  - 모든 Estimator는 fit() 함수를 가지고 있음
    - fit()은 DataFrame을 받아 Model을 반환
    - 반환된 Model은 Transformer 기능을 함
  - 예시
    ```Python
    lr = LinearRegression()   
    model = lr.fit(train_data)   # Estimator 
    model.transform(test_data)   # Transformer 
    ```
    
### 3) Evaluator

  - metric을 기반으로 모델의 성능을 평가하는 기능
    - ex) Root Mean Squared Error (RMSE)
  - 모델을 여러개 만들어서, 성능을 평가한 후 가장 좋은 모델을 뽑는 방식으로 튜닝 자동화
  - 예시
    - BinaryClassificationEvaluator
    - CrossValidator
