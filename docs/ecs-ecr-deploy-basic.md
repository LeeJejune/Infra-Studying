name: ecs-ecr-actions

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
- name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws_access_key: AWS_ACCESS_KEY_ID # IAM 키
          aws_secret_key: AWS_SECRET_ACCESS_KEY # IAM 비밀 키
          aws-region: ap-northeast-2
~~~
- AWS ECR의 푸쉬를 위해 인증 및 로그인을 해준다.

~~~
 - name: Login to Amazon ECR
 id: login-ecr
 uses: aws-actions/amazon-ecr-login@v1
~~~
- ECR의 로그인을 해준다.

~~~
- name: Build, tag, and push image to Amazon ECR # ECR Build 및 Push
  id: build-image
  env:
    ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    ECR_REPOSITORY: ECR_REPOSITORY # ECR repository 이름
    IMAGE_TAG: ${{ github.sha }} # 깃 액션 버저닝
    run: |
      docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
      docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
      echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
~~~
- 우리가 생성하고 정의한 ECR을 활용하여 도커 이미지를 푸쉬 해준다.

~~~
- name: ECS task definition
  id: task-def
  uses: aws-actions/amazon-ecs-render-task-definition@v1
  with:
    task-definition: .aws/task-definition.json # 우리가 정의하는 ECS의 테스크 구성
    container-name: jejune-test
    image: ${{ steps.build-image.outputs.image }}
~~~
- 우리가 ECS를 활용해서 컨테이너를 관리하는 것을 테스크와 서비스를 사용한다. 
- task-definition은 AWS GUI 혹은 Json 파일로 정의할 수 있다. 그 안에는 우리의 ECR 정보 및 테스크, IAM이 정의되어 있다.

~~~
- name: Deploy Amazon ECS task definition
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: ${{ steps.task-def.outputs.task-definition }}
    service: ECS-SERVICE # 정의한 ECS의 Service
    cluster: ECS-CLUSTER # 정의한 ECS의 Cluster
    wait-for-service-stability: true
~~~
- 이제는 ECS를 통해 우리가 원하는 동작을 Deploy하는 단계이다.
- 우리가 정의했던 ECR Service, Task의 이름을 기입해준다.