# 병렬처리와 분산처리

<br/>

## 1. 병렬처리와 분산처리

||<center>병렬처리</center>|<center>분산처리 (분산된 환경에서의 병렬처리)</center>|
|:---|:---|:---|
|개념그림|    <center>![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/e07c8f93-5de6-4997-a6cc-db079d058998)</center>|    <center>![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/50336d3e-f184-4d4f-ad08-ec9aa20163aa)</center>|
|설명|1. 데이터를 여러개로 쪼개고 <br/> 2. 여러 쓰레드에서 각자 task를 적용한 뒤 <br/> 3. 각자 만들어진 결과값을 합침|1. 데이터를 여러깨로 쪼개 **여러 노드로 보냄** <br/> 2. 여러 노드에서 각각 독립적으로 task를 적용 <br/> 3. 각자 만들어진 결과값을 합침|

<br/>

* 분산된 환경에서의 병렬처리시 Spark의 강점

  - 분산된 환경에서는 노드간 통신 등 신경써야할 것이 늘어남
  - 그러나 Spark에서는 분산된 환경에서도 일반적인 병렬처리를 하듯 코드를 짤 수 있음
    - RDD를 통해 분산된 환경에서 데이터 병렬 모델을 구현해 추상화 시켜주기 때문


<br/>

## 2. 분산처리시 속도 (Latency) 문제

* 분산처리시 신경써야 할 문제들
  - 부분 실패: 노드 몇개가 프로그램과 상관없는 이유로 실패
  - 속도 문제: 많은 네트워크 통신을 필요로 하는 작업일수록 속도가 저하됨 
  
    - 예시
        ~~~python
        RDD.map(A).filter(B).reduceByKey(C).take(100)
        RDD.map(A).reduceByKey(C).filter(B).take(100)
        ~~~
      - reduceByKey는 여러 노드에서 데이터를 불러오기 때문에 통신을 필요로 함
      - 전자는 filter를 거친 후 reduceByKey를 수행 vs 후자는 reduceByKey를 거친 후 filter를 수행
      - **전자의 경우 한 번 데이터가 줄어든 상태에서 통신을 하기 때문에 속도가 훨씬 더 빠름!**
  - 이렇듯 Spark의 작업이 뒷단에서 어떻게 돌아갈지 상상하며 코드를 짜는 것이 중요함
