pipeline {
    agent any
    environment {
        webhookSecretIdKey = 'p3-daewoon-sonarcloud-test-user-service-jenkins-webhook'
    }
    stages {
        stage ('User-Service - Run Maven Tests, Lint, and Submit to SonarQube'){ 
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh "docker run \
                    --user \"\$(id -u):\$(id -g)\" \
                    --rm \
                    -v `pwd`:/container/directory \
                    -w /container/directory \
                    maven:3.8.5-openjdk-8-slim \
                    mvn test clean" 

                    sh "docker run \
                    --user \"\$(id -u):\$(id -g)\" \
                    --rm \
                    --env SONAR_TOKEN=${env.SONAR_AUTH_TOKEN} \
                    -v `pwd`:/container/directory \
                    -w /container/directory \
                    maven:3.8.5-openjdk-11-slim \
                    mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                    -Dsonar.projectKey=p3-daewoon-test-user-service"
                }
            }
        }
        stage("User-Service - Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true, webhookSecretId: "${webhookSecretIdKey}"
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