## 빈스톡 도커로 맛보기

~~~
- name: Set up JDK 17
  uses: actions/setup-java@v2
  with:
    java-version: '17' 
    distribution: 'zulu'
~~~
- 우리가 사용하는 자바를 세팅 해준다.

~~~
- name: Grant execute permission for gradlew
  run: chmod +x gradlew

- name: Build with Gradle
  run: ./gradlew build
~~~
- 사용중인 (필자는 스프링 사용 및 Gradle 사용중) 프로젝트의 빌드 과정을 거친다.

~~~
- name: Docker build
  run: |
    docker login -u 도커허브 유저네임 -p 도커허브 토큰
    docker build -t 이미지명 .
    docker tag admin 사용자명/이미지명:태그명
    docker push 사용자명/이미지명:태그
~~~
- 빌드 과정을 거친 후 우리가 사용중인 도커 허브 및 컨테이너에 이미지를 푸쉬 한다!
- 여기서 도커를 직접 빌드하여 도커 허브에 푸쉬하지 않고 AWS ECR을 사용할 수 있다! (선택 요소)

~~~
- name: Beanstalk Deploy
  uses: einaregilsson/beanstalk-deploy@v20
  with:
    aws_access_key: AWS_ACCESS_KEY_ID # IAM 키
    aws_secret_key: AWS_SECRET_ACCESS_KEY # IAM 비밀 키
    application_name: example # beanstalk application 이름
    environment_name: example-env # beanstalk application 환경 이름
    version_label: "github-action--${{ steps.format-time.outputs.replaced }}"
    region: ap-northeast-2
    deployment_package: Dockerrun.aws.json # 도커 포트 설정
~~~
- 이제 우리가 설정한 빈스톡 애플리케이션 및 환경에 배포를 하는 과정이다.
- 빈스톡에 배포할 권한을 가진 IAM을 만들어주고 깃 액션 시크릿에 해당 내용을 기술해준다.
- 이제 액션 yml에서 이를 지정해주고 빈스톡 애플리케이션 및 환경을 설정해준다.
- 그리고 region 설정을 꼭 해줘야한다!
- (필자는 EC2 Profile을 설정하지 않았어서 트래블 슈팅을 진행했다. 꼭 설정해주기!)
- 마지막으로 도커 사용하기 때문에 Dockerrun.aws.json을 작성해 이를 프로젝트에 포함시켜주기!
  - 여기서 Dockerrun.aws.json을 설정해줘야 포트 설정 및 어떤 이미지를 사용할건지를 기술해줄 수 있다.