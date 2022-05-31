pipeline {
    agent any
    environment {
        SONAR_HOST_URL = 'http://34.130.147.227:9000' /*docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest*/

        //Can create secret text in jenkins configuration for webhooks. In sonarqube will have to specify webhook if you do.
        SONAR_USER_SERVICE_PROJECT_KEY = 'p3-reverse-user-service'
    }
    stages {
        stage ('User-Service - Run Maven Tests and Submit to SonarQube'){ 
            steps {
                withSonarQubeEnv('sonarqube-p3-test') {
                    sh "cd docker run \
                    --user \"\$(id -u):\$(id -g)\" \
                    --rm \
                    -v `pwd`:/container/directory \
                    -w /container/directory \
                    maven:3.8.5-openjdk-8-slim \
                    mvn test"

                    sh "cd User-Service; rm .git; docker run \
                    --user \"\$(id -u):\$(id -g)\" \
                    --rm \
                    -v `pwd`:/container/directory \
                    -w /container/directory \
                    maven:3.8.5-openjdk-11-slim \
                    mvn clean verify sonar:sonar \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_AUTH_TOKEN} \
                    -Dsonar.projectKey=${SONAR_USER_SERVICE_PROJECT_KEY}"
                    // -Dsonar.scm.exclusions.disabled=true \
                    // -Dsonar.scm.disabled=True"
                }
            }
        }
        stage("User-Service - Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true, webhookSecretId: 'sonar-userservice-webhook'
                }
            }
        }
        stage ('Docker Build'){
            steps {
                sh 'docker build -t reverse-user-service .'
            }
        }
        // stage ('Docker Push'){
        //     steps {
        //         sh 'docker tag reverse-user-service northamerica-northeast2-docker.pkg.dev/devops-javasre/p3/reverse-user-service'

        //         sh 'docker push northamerica-northeast2-docker.pkg.dev/devops-javasre/p3/reverse-user-service'
        //     }
        // }
    }
}