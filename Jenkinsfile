stages:
  - prepare          # List of stages for jobs, and their order of execution
  - compile_test
  - security
  - build
  - docker_build_push
  - deploy
 
  

prepare-tool:       # This job runs in the build stage, which runs first.
  stage: prepare
  script:
    - mvn  --version
    - java --version
  tags:
  - self-hosted

compile:       # This job runs in the build stage, which runs first.
  stage: compile_test
  script:
    - mvn compile
  tags: 
  - self-hosted

    
test:       # This job runs in the build stage, which runs first.
  stage: compile_test
  script:
    - mvn test
  tags: 
  - self-hosted

trivy_fs_scan:       # This job runs in the build stage, which runs first.
  stage: security
  script:
    - trivy fs --format table -o trivy-fs-report.html .
  tags: 
  - self-hosted


sonarqube-check:
  stage: security
  image: adijaiswal/sonar-maven:latest
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  
    GIT_DEPTH: "0"  
  cache:
    key: ${CI_JOB_NAME}
    paths:
      - .sonar/cache
  script: 
  - sonar-scanner
   -Dsonar.projectKey=batch-4840424_game_AZBIztabnxHIeySEwl_E
   -Dsonar.java.binaries=.

  allow_failure: true
  only:
    - main


build_application:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - mvn package
  tags: 
  - self-hosted


build_docker_image:       # This job runs in the build stage, which runs first.
  stage: docker_build_push
  script:
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - mvn package
    - docker build -t adijaiswal/bgame:latest .
  tags: 
  - self-hosted


push_docker_image:       # This job runs in the build stage, which runs first.
  stage: docker_build_push
  script:
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - docker push adijaiswal/bgame:latest 
  tags: 
  - self-hosted


k8s-deploy:
  stage: deploy
  image: lachlanevenson/k8s-kubectl:1.18.0
  variables:
     KUBECONFIG_PATH: /home/ubuntu/config
  before_script:
    - mkdir -p $(dirname "$KUBECONFIG_PATH")
    - echo "$KUBECONFIG_CONTENT" | base64 -d   > "$KUBECONFIG_PATH"
    - export KUBECONFIG="$KUBECONFIG_PATH"
  script:
    - kubectl apply -f deployment-service.yaml
  only:
    - main
  tags:
    - self-hosted
