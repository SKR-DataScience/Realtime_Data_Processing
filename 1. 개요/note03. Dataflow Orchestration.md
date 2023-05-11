# 개요3. Dataflow Orchestration

## Orchestration 개요

- Orchestration : '지휘'
  - 오케스트라처럼 데이터 태스크를 지휘
    1) 태스크 스케쥴링
    2) 분산 실행
    3) 태스크간 의존성 관리

- Orchestration의 필요성
  - 서비스가 커지면서, 데이터 플랫폼의 복잡도 증가
  - 데이터가 사용자와 직접 연관되는 경우가 늘어남 (workflow가 망가지면 서비스도 망가짐)
  - 태스크 하나하나가 중요해졌고, 태스크간 의존성도 생김

- Dataflow Orchestration의 대표적인 도구 : **Apache Airflow**
  - 자세한 내용은 앞으로 강의에서 정리...
