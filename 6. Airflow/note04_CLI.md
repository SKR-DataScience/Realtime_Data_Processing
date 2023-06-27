# Airflow CLI

- 에어플로우에서는 다음과 같은 커맨드 사용 가능

    ```bash
    > airflow -h

    # 다양한 커맨드들
    Usage: airflow [-h] GROUP_OR_COMMAND ...

    Positional Arguments:
    GROUP_OR_COMMAND
        
        Groups: # 번들 사용
        celery         Celery components
        config         View configuration
        connections    Manage connections
        dags           Manage DAGs # 대그 관리
        db             Database operations # 디비 관리
        jobs           Manage jobs
        kubernetes     Tools to help run the KubernetesExecutor
        pools          Manage pools
        providers      Display providers
        roles          Manage roles
        tasks          Manage tasks
        users          Manage users
        variables      Manage variables
        
        Commands: # 일회성 사용
        cheat-sheet    Display cheat sheet 
        dag-processor  Start a standalone Dag Processor instance
        info           Show information about current Airflow and environment # 현 환경정보
        kerberos       Start a kerberos ticket renewer
        plugins        Dump information about loaded plugins
        rotate-fernet-key
                        Rotate encrypted connection credentials and variables
        scheduler      Start a scheduler instance
        standalone     Run an all-in-one copy of Airflow
        sync-perm      Update permissions for existing roles and optionally DAGs
        triggerer      Start a triggerer instance
        version        Show the version
        webserver      Start a Airflow webserver instance # 웹서버 시작

    ```

    ## user 생성

    - user 생성(로그인 아이디, 비밀번호 지정)
        ``` bash
        > airflow users create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin
        ```
    - 두 번째 유저를 생성하려면
        ``` bash
        > airflow users create -r Admin -u admin2 -e admin2@gmail.com -f yuri -l kim -p admin2
        ```
    - airflow users list로 현재까지 생성된 유저 확인
        <center><img src="./fig4_1.png" width="100%"></center>
    - 참고. airflow users 명령어로 사용할 수 있는 기능
        ```bash
        > airflow users -h

        Usage: airflow users [-h] COMMAND ...

        Manage users

        Positional Arguments:
        COMMAND
            add-role   Add role to a user
            create     Create a user
            delete     Delete a user
            export     Export all users
            import     Import users
            list       List users
            remove-role
                    Remove role from a user

        Optional Arguments:
        -h, --help   show this help message and exit
        ```


## airflow 페이지 열기

- bash
    ```bash
    airflow webserver
    ```
    - 터미널에 다음과 같은 내용이 뜨면 localhost 접속을 진행
        <center><img src="./fig4_1.png" width="100%"></center>

- web
    ```bash
    localhost:8080
    localhost:8080/home # 내 경우는 이미 제플린이 localhost:8080 사용중이어서 /home을 붙여야 airflow가 떴다. 
    ```
    
    - 위에서 설정한 username과 password인 admin으로 접속한다.
        <center><img src="./fig4_2.png" width="100%"></center>

    - 접속하면 다음과 같은 워크플로우 화면이 나온다.
        <center><img src="./fig4_3.png" width="100%"></center>


- 참고: 포트 충돌
    - 포트가 완전히 안닫히면 'The webserver is already running under PID 2221.'와 같은 에러가 남
        ```
        # 완전히 안닫힌 pid 종료
        kill -9 $(lsof -t -i:8080)
        ```