# GitHub Actions 사용하여 CI/CD 적용해보기 


용어
==

Github Action 의 개념 용어로는 Workflow, Event, Job, Step, Action, Runner 가 있다.

*   **Workflow**
    *   Github Action 의 최상위 개념
    *   Repository의 .github/workflows 폴더 아래에 저장
    *   하나 이상의 Job으로 구성되고, Push나 PR과 같은 Event에 또는 특정 시간대에 실행 될 수 있는 자동화된 프로세스
*   **Event**
    *   Workflow를 Trigger(실행)하는 특정 활동이나 규칙
    *   특정 활동이란 Push, Pull Request, Commit 특정시간(Cron) 등을 의미
    *   특정 행동이 아닌,webhook ([Repository Dispatch Webhook](https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event))을 사용하면 Github 외부에서 발생한 이벤트에 의해서도 Workflow를 실행 가능
*   **Runner**
    *   Runner란 Workflow 안의 Job을 실행시키기 위한 애플리케이션 머신
    *   Github에서 호스팅해주는 Github-hosted runner와 직접 호스팅하는 Self-hosted runner로 구분
    *   Github-hosted runner는 Azure의 Standard\_DS2\_v2로 vCPU 2, 메모리 7GB, 임시 스토리지 14GB
*   **Job**
    *   Job은 동일한 Runner에서 실행 되는 여러 Step으로 구성되고, 가상 환경의 인스턴스에서 실행
    *   다른 Job에 의존 관계를 가질 수 있고, 독립적으로 병렬 실행도 가능
    *   Job 의 관계를 주어 Test Job 은 항상 Build Job이 성공해야만 동작하는등 의 행위를 할수있다.
*   **Step**
    *   Task들의 집합으로, 커맨드를 날리거나 action을 실행할 수 있음
    *   하나의 Job 내에서 각각의 Step은 다양한 Task로 인해 생성된 데이터를 공유할 수 있다.
*   **Action**
    *   Workflow의 가장 작은 블럭(smallest portable building block)
    *   Job을 만들기 위해 Step들을 연결할 수 있음
    *   재사용이 가능한 컴포넌트
    *   개인적으로 만든 Action을 사용할 수도 있고, Marketplace에 있는 공용 Action을 사용할 수도 있음
        *   [Github Marketplace](https://github.com/marketplace?type=actions)와 [Github Actions Repository](https://github.com/actions/)에서 확인 가능

제한
==

*   Pree  
    *   Storage 한도 500MB, 월에 실행 시간 2,000분
*   Pro  
    *   Storage 한도 1GB, 월에 실행 시간 3,000분

[https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)

적용하기
====

.github/workflows 디렉토리를 생성후 .yml 파일을 생성해도되지만

github가 다양한 템플릿을 제공하기에 이것 사용하자

이중 [set up a workflow yourself](https://github.com/k4keye/VisitKnowledge/new/master?filename=.github%2Fworkflows%2Fmain.yml&workflow_template=blank) 를 사용할것이다

![](68fd02bf-636e-446b-a67e-50b2131ef8aa.png)

YML 작성방법
========

yml 을 특정 키워드를 이용하여 특정 행위를 할수있게한다.

다양한 예약어들이 존재하지만 필수적인 예약어만 알아본다면

먼저 최상단 부분에는 action에 대한 정보를 작성하며

*   name : action 의 이름을 명시
*   on : 특정 브랜치에 어떤 이벤트로 동작할것인지에 대한 명시
    *   예) master 브랜치에 push
    *   예) develop 브랜치에 PR
*   jobs : action에서 동작할 하나 이상의 job 목록
    *   job 이름과 Runner의 실행 환경을 지정한다.

실제 동작의 핵심이 되는 steps 의 키워드로는

*   name: step의 이름으로 workflow 에 표시됨
*   run: 명령을 수행
*   uses: 다른 action을 불러와서 수행

등이 존재한다.

uses 를 통해 다양한 action을 불러와서 수행할수있는데

github에는 action 의 marketplace 가 있으며 여기서 필요한 action들을 사용할수있다.

[GitHub Marketplace: actions to improve your workflow](https://github.com/marketplace?type=actions)

이제 점진적으로 기능을 추가해가면서 어떻게 사용하는지 알아보자.

master push 혹은 PR 시 echo 수행
===========================

    name: CI/CD
    on:
      push:
        branches: [master]
      pull_request:
        branches: [master]
    jobs:
      build:
        runs-on: ubuntu-latest
    
        steps:
          - name: Run Master Branch CI
            run: echo Hello, CI!
    

만약 브랜치 구분없이 동작하고싶다면

    on: [push, pull_request]
    

위와같이 작성할 수 있다.

master push 시 빌드 후 echo 수행
==========================

    //생략 ----
    
    - name: checkout action
      uses: actions/checkout@v3
    
    - name: setup JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'zulu'
      
    - name: Grant execute permission 
      run: chmod +x gradlew
      
    - name: gradlew build
      run: ./gradlew clean build --exclude-task test
    
    - name: Run Master Branch CI
      run: echo Hello, CI!
    

이곳에서는 빌드를 할 master로 이동하기위해, 그리고 빌드를 할 환경을 구성하기위한 java 를

github action marketplace에서 가지고와서 사용하였다.

master push 시 빌드 후 slack 메시지 보내기
================================

일단 slack 에 메시지를 보내기위해 github action marketplace 에서 찾아보니

slack api web hooks 를 사용하여 메시지를 보내는 action이 있어 사용하려한다.

물론 slack bot으로도 진행가능.

![](191ee826-4f94-4042-be98-044413725c7f.png)

일단 슬랙에서 webHooks 를 설치하고

![](66601edc-7bfa-4f25-845e-1ce6e94a52b8.png)

webHooks URL 을 획득한다.

그리고 해당 URL 을 GitHubAction 에서 사용하려고 yml 파일에 작성하게되면

중요한 URL 이 노출되는것이기에 이를 감출 필요가있다.

![](5a86fe59-34a6-417e-b70b-5613dfcda480.png)

repository - settion - secrets - new repository secret

에서 webHooks URL 을 추가한다.

이렇게 숨긴 secret 값은

${{ secrets.SECRET-NAME }} 로 사용할수있다.

    //생략 ....
    
    - name: slack send message
      uses: slackapi/slack-github-action@v1.23.0
      with:
        payload: |
          {
            "text": "GitHub Action build result: ${{ job.status }}\\n${{ github.event.pull_request.html_url || github.event.head_commit.url }}",
            "blocks": [
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "GitHub Action build result: ${{ job.status }}\\n${{ github.event.pull_request.html_url || github.event.head_commit.url }}"
                }
              }
            ]
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACKTESTHOOK }}
    

이제 master에 push 를 하면

![](876de6da-60a8-4b9c-90e2-8702168e8886.png)

위와같은 메시지를 받을수있다.

Docker hub 업로드
==============

먼저 Dockerfile를 작성해준다

    FROM adoptopenjdk/openjdk11
    
    COPY ./build/libs/<project-name>-<version>-SNAPSHOT.jar app.jar
    
    ENTRYPOINT ["java", "-jar", "app.jar"]
    

action yml에 docker 이미지를 빌드하여 업로드 하는 과정을 추가한다.

    - name: Docker Image Build
      run: docker build -t <docker hub nickname>/<product-name> .
    
    - name: Docker Hub Login
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_HUB_ID }}
        password: ${{ secrets.DOCKER_HUB_TOKEN }}
    
    - name: Docker Hub push
      run: docker push <docker hub nickname>/<product-name>
    

Build Job, Deploy Job 분리
========================

하나의 job에서 모든 기능을 수행하는것이 아닌

build job가 완료되면 deploy job이 실행되도록 작성하려한다.

먼저 job의 순서가 중요하기에

needs 예약어를 통해 build job이 완료된후에 deploy job이 실행되도록한다.

    jobs:
      build:
        steps:
    				//생략...
         
      deploy:
        needs: build
        steps:
    				//생략...
    

이제 위에서 부터 진행해온 코드를 분리만 하면될것같지만

각 job은 다른 환경이기에

build job에서 만들어진 jar를 deploy에서 그대로 사용할수없는 문제가 발생하니

build job의 jar를 업로드 하고 deploy는 해당 jar를 다운로드 하여 사용해야한다.

build job 는 upload-artifact 을 통해 필요한 jar, dockerfile 를 업로드하고

deploy job 는 download-artifact 을 통해 필요한 jar, dockerfile 를 다운로드 할것이다.

    jobs:
      build:
        steps:
          - name: upload jar
            uses: actions/upload-artifact@v1
            with:
              name: build
              path: ./build
    
          - name: upload dockerfile
            uses: actions/upload-artifact@v1
            with:
              name: dockerfile
              path: ./Dockerfile
         
      deploy:
        needs: build
        steps:
          - name: download jar
            uses: actions/download-artifact@v1
            with:
              name: build
    
          - name: download dockerfile
            uses: actions/download-artifact@v1
            with:
              name: dockerfile

동작 제어
=====

1\. 특정 커밋메시지면 동작 안하게 하기
-----------------------

    jobs:
      build:
    	 ...
      deploy:
         needs: build
         ...

위와같이 동작하던 job에서 build 를 if 절 을 사용하여 특정 커밋메시지가 존재하면 동작하지 않게 할것이다

    jobs:
      build:
        if: ${{ !contains(github.event.head_commit.message, '[master action skip]') }}
    	...
      deploy:
        needs: build
        ...

커밋 메시지 안에 "\[master action skip\]" 문구가 들어가면 동작하지 않게 build job에 if 절을 추가한다.

![](9a6256c7-70d6-4a77-bf73-3b13a69ab895.png)

2.  상위 step이 실패해도 동작하기
----------------------

조건의 failure 와 always 를 사용하여 위의 step 가 실패했을때 동작하거나

무조건 실행되도록 할수있다.

    steps:
    	- name: fail step
    	  run: fail...
    	
    	- name: always step
    	  if: ${{ always() }}
    		run: echo always step!
    	
    	- name: failure step
    	  if: ${{ failure() }}
    		run: echo failure step!
    		
    

3\. 상위 job  성공시, 실패시 동작하도록 하기
-----------------------------

needs를 통해 순서가 정해진 job에서

자신 앞에 호출된 job의 결과를 얻을수있다.

    jobs:
    	deploy:
    	    needs: docker
    	    runs-on: ubuntu-latest
    	    steps:
    	      - name: deploy start
    	        run: echo deploy start
    	
    	report-deploy-success:
    	  needs: deploy
    	  runs-on: ubuntu-latest
    	  if: ${{ needs.deploy.result == 'success' }}
    	  steps:
    	    - name: slack send message
    	      run: echo ${{ needs.deploy.result }}
    	
    	report-deploy-failure:
    	  needs: deploy
    	  runs-on: ubuntu-latest
    	  if: ${{ needs.deploy.result == 'failure' }}
    	  steps:
    	    - name: slack send message
    	      run: echo ${{ needs.deploy.result }}
    

전체 코드

    name: CI/CD
    
    on:
    
      push:
        branches: [master]
      pull_request:
        branches: [master]
    
    jobs:
    
      report-workflow:
        runs-on: ubuntu-latest
        steps:
          - name: event slack send message
            run: echo report-job-state!
    
    
      build:
        if: ${{ !contains(github.event.head_commit.message, '[skip]') }}
        runs-on: ubuntu-latest
    
    
        steps:
          - name: Run Master Branch CI
            run: echo Hello, CI!
    
         - name: checkout action
            uses: actions/checkout@v3
    
          - name: setup JDK 11
            uses: actions/setup-java@v3
            with:
              java-version: '11'
              distribution: 'zulu'
    
    
          - name: Grant execute permission
            run: chmod +x gradlew
    
          - name: gradlew build
            run: ./gradlew clean build -Pprofile=prod --exclude-task test
    
          - name: upload jar
            uses: actions/upload-artifact@v1
            with:
              name: build
              path: ./build
    
          - name: upload dockerfile
            uses: actions/upload-artifact@v1
            with:
              name: dockerfile
              path: ./Dockerfile
    
      report-build-success:
        needs: build
        runs-on: ubuntu-latest
        if: ${{ needs.build.result == 'success' }}
        steps:
          - name: slack send message
            run: echo ${{ needs.build.result }}
    
      report-build-failure:
        needs: build
        runs-on: ubuntu-latest
        if: ${{ needs.build.result == 'failure' }}
        steps:
          - name: slack send message
            uses: slackapi/slack-github-action@v1.23.0
            with:
              channel-id: ${{ secrets.SLACK_BOT_CHANNEL }}
              slack-message: "[Master Branch] CI/CD build job failure"
            env:
              SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
    
    
      docker:
        needs: build
        runs-on: ubuntu-latest
        steps:
    
          - name: download jar
            uses: actions/download-artifact@v1
            with:
              name: build
    
          - name: download dockerfile
            uses: actions/download-artifact@v1
            with:
              name: dockerfile
    
    
          - name: echo build/libs dir
            run: ls -al ./build/libs
    
          - name: move dockerfile
            run: mv dockerfile/Dockerfile .
    
          - name: Docker Image Build
            run: docker build -t k4keye/vkestrel .
    
          - name: Docker Hub Login
            uses: docker/login-action@v2
            with:
              username: ${{ secrets.DOCKER_HUB_ID }}
              password: ${{ secrets.DOCKER_HUB_TOKEN }}
    
          - name: Docker Hub push
            run: docker push k4keye/vkestrel
    
          - name: slack send message
            if: ${{ failure() }}
            uses: slackapi/slack-github-action@v1.23.0
            with:
              channel-id: ${{ secrets.SLACK_BOT_CHANNEL }}
              slack-message: "[Master Branch] CI/CD docker job ${{ job.status }}"
            env:
              SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
    
      report-docker-success:
        needs: docker
        runs-on: ubuntu-latest
        if: ${{ needs.docker.result == 'success' }}
        steps:
          - name: slack send message
            run: echo ${{ needs.docker.result }}
    
      report-docker-failure:
        needs: docker
        runs-on: ubuntu-latest
        if: ${{ needs.docker.result == 'failure' }}
        steps:
          - name: slack send message
            uses: slackapi/slack-github-action@v1.23.0
            with:
              channel-id: ${{ secrets.SLACK_BOT_CHANNEL }}
              slack-message: "[Master Branch] CI/CD docker job failure"
            env:
              SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
    
      deploy:
        needs: docker
        runs-on: ubuntu-latest
        steps:
    
          - name: deploy start
            run: echo deploy start
          - name: deploy server accessq
            uses: appleboy/ssh-action@v0.1.6
            with:
              host: ${{ secrets.WAS_HOST }}
              username: ${{ secrets.WAS_USERNAME }}
              password: ${{ secrets.WAS_PASSWORD }}
              port: ${{ secrets.WAS_SSH_PORT }}
              script: |
                docker stop vkestrel
                docker rm vkestrel
                docker pull k4keye/vkestrel
                docker run -d -p 8791:8791 --name vkestrel k4keye/vkestrel
    
    
      report-deploy-success:
        needs: deploy
        runs-on: ubuntu-latest
        if: ${{ needs.deploy.result == 'success' }}
        steps:
          - name: slack send message
            run: echo ${{ needs.deploy.result }}
    
      report-deploy-failure:
        needs: deploy
        runs-on: ubuntu-latest
        if: ${{ needs.deploy.result == 'failure' }}
        steps:
          - name: slack send message
            uses: slackapi/slack-github-action@v1.23.0
            with:
              channel-id: ${{ secrets.SLACK_BOT_CHANNEL }}
              slack-message: "[Master Branch] CI/CD deploy job failure"
            env:
              SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

![](796ebcca-7ff4-422b-a0e9-158c000c3f19.png)