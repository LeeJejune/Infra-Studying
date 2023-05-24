## 깃 액션 맛보기

~~~
name: Github Action Jejune First Example
run-name: Hello World!!
on: [push]
    jobs:
    hello_world_github_actions:
        runs-on: ubuntu-latest
        steps:
        - run: echo "Hello world"
~~~

깃 액션을 사용하기 위한 가장 간단한 예제입니다!

- name : 해당 액션의 이름 지정
- on: push, commit 시 workflow를 트리거한다
  - pull_request, push와 같이 이벤트에 대한 트리거를 걸 수 있다.
- jobs: 빌드 대상 인스턴스 내의 여러 개 step들을 그룹으로 실행하는 작업단위이다.
- steps: 순차적으로 명령어를 수행합니다. run/uses가 있으며 uses는 정의한 명령어, run은 실행 스크립트이다.