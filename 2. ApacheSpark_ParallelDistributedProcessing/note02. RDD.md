# 개요2. Resilient Distributed Dataset (RDD)

<br/>

## 1. RDD란?

* Resilient Distributed Dataset : '탄력적 분산 데이터셋'
  - 'Resilient'(탄력적or회복력을 가진) -> 데이터를 처리하는 과정에서 문제가 발생하더라도 스스로 복구 가능

* 스파크의 기본 데이터 구조
  - 스파크의 모든 작업은 새로운 RDD 생성 / 존재하는 RDD를 변형 / RDD에서의 결과 연산을 표현하는 것임

<br/>

## 2. RDD의 특징

**1) 데이터 추상화**
  - 실제 (대용량의) 데이터는 클러스터의 여러 노드에 흩어져 있지만, 하나의 파일인 것처럼 사용 가능
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/c3242430-4b35-4b44-95d0-b35426ef617c)
  
  - 파이썬 코드에서는...   
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/2b56afee-da22-4a41-9fa8-6b914068441c)
    - 여러 노드에 담겨있는 데이터를 'lines'라는 객체 하나로 다룰 수 있음

<br/>

**2) Resilient & Immutable (탄력성, 불변성)**
    
  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/f90cbe3b-1e87-472b-b6ec-1a48257b1435)

  - RDD1이 변환을 거치면, RDD1이 바뀌는게 아니라 새로운 RDD2가 만들어짐 (Immutable)
  - 변환을 거칠 때마다, 연산의 기록이 남음 -> 하나의 비순환 그래프(Acylic Graph)로 그릴 수 있게 됨
  - 덕분에 문제가 생길 경우, 쉽게 이전 RDD로 돌아갈 수 있음
  - Node 1에서 연산 중 문제가 생기면, 다시 복원 후 Node 2에서 연산하면 됨 (Resilient)

<br/>

**3) Type-safe**

  - 컴파일시 Type을 판별할 수 있어 문제를 일찍 발견할 수 있음 (개발자 친화적)

<br/>

**4) Unstructured / Structured Data를 모두 담을 수 있음**

  - Unstructured data : 텍스트 / 로그 / 자연어 / ...
  - Structured data : RDB / DataFrame / ...

<br/>

**5) Lazy Execution**

  - '게으른 연산': 결과가 필요할 때까지 연산을 하지 않음

  - Spark Operation = Transformation(변환) + Action(액션)

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/05ebe458-a3ed-4ec2-9080-5e94a736b740)
  
  - 액션을 할 때까지 변환은 실행되지 않음 (액션을 만난 시점에 이전 연산들이 전부 실행됨) 

<br/>

## 3. RDD를 쓰는 이유

- 유연성
- 짧은 코드로 할 수 있는게 많음
- 개발할 때, '무엇'보다는 '어떻게'에 대해 더 생각하게 함(how-to)
  - Lazy Execution 덕분에 데이터가 어떻게 변환될지 고려하게 됨
  - '데이터가 지나갈 길'을 닦는 느낌...
