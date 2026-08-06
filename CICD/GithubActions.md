# Github Actions 기본 문법 정리

## 파일 생성
> .github 디렉토리 → workflows 디렉토리 → deploy.yml 파일

```yml
# 이 하나의 파일을 workflow라고 한다.
name: Github Actions 실행

# Event: 실행되는 시점을 설정
# main이라는 브랜치에 push 될 때 아래 workflow를 실행
on:
  push:
    branches:
      - main

# 1개의 workflow는 1개 이상의 Job으로 구성된다.
# 여러 Job은 기본적으로 병렬적으로 수행된다.
jobs:
  My-Deploy-Job: # Job을 식별하기 위한 Id
    runs-on: ubuntu-latest # ubuntu 환경에서 가장 최신 버전을 쓰겠다고 설정

    # Step: 특정 작업을 수행하는 가장 작은 단위
    # Job은 여러 Step들로 구성되어 있다.
    steps: 
      - name: Hello World 찍기
        run: echo "Hello World"

      - name: 여러 명령어 문장 작성하기
        run: |
          echo "Good"
          echo "Morning"

      - name: Github Actions 자체에 저장되어 있는 변수 사용해보기
        run: |
          echo $GITHUB_SHA
          echo $GITHUB_REPOSITORY

      - name: 아무한테도 노출이 되면 안 되는 값
        run: |
          echo ${{ secrets.MY_NAME }}    # result: ***
          echo ${{ secrets.MY_HOBBY }}   # result: ***
```

## Actions 돌아가는 방식
> Repository → Actions → Commit Message로 리스트가 생김 → workflow에 있는 코드가 돌아감

## 환경 변수 설정하는 법
> Settings → Secrets and variables → Actions
