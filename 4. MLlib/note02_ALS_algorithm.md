# 추천 알고리즘 - ALS

## 1. ALS(Alternating Least Squares)

* 협업 필터링(Collaborative Filtering) 기반 추천 알고리즘의 한 종류   

  * 협업 필터링?   

    ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/12e374ff-3360-43ac-bbd3-ed33cef79b3c)
    - 유저의 과거 행동 데이터(평점 부여)만 가지고 추천
    - 유저A와 비슷한 평점 부여 패턴을 보인 유저B가 높은 평점을 줬던 item(Casablanca)을 추천

  * 실제 데이터는 아래와 같음

    * 한 유저가 볼 수 있는 영화는 굉장히 많고, 따라서 빈 칸이 많음    
    ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/b2fbc4b8-ba8c-4277-8c7e-77147fe2899f) 

    * 빈 칸에 들어갈 영화 평점을 예측하여 값을 정렬 -> 유저별로 가장 높은 평점으로 예측된 영화를 추천!
    ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/a51ea5a4-00ef-4127-bde7-d20994271063)


<br/>

* **그래서 ALS는 무엇인가**   

  ![image](https://github.com/SKR-DataScience/Realtime_Data_Processing/assets/55543156/46860048-f563-490b-8084-79f056dcf3c2)  

  - 빈칸이 많은 Rating Matrix는 2개의 Matrix로 나눌 수 있음 (User Matrix / Item Matrix)
  - ALS는 두 Matrix 중 하나를 고정시키고, 다른 하나의 Matrix를 순차적으로 반복하면서 최적화하는 방식
    1) Item/User Matrix의 값이 랜덤한 값으로 채워짐
    2) Item Matrix를 고정시키고 User Matrix를 최적화 (두 Matrix의 값의 곱이 Rating Matrix의 값과 비슷해지도록)
    3) 그 다음 User Matrix를 고정시키고 Item Matrix를 최적화
    4) 위 과정을 반복 -> 최적화된 값이 실제 Rating Matrix의 값과 가장 가까워질 때 최적화 끝   
      -> 이 때 빈칸에 채워진 예측값들이 우리의 추천 결과값이 됨
     
