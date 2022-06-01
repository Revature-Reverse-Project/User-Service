pipeline {
    agent any
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
                    -v `pwd`:/container/directory \
                    -w /container/directory \
                    maven:3.8.5-openjdk-11-slim \
                    mvn verify sonar:sonar \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_AUTH_TOKEN} \
                    -Dsonar.projectKey=${SONAR_USER_SERVICE_PROJECT_KEY}
                    mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                    -Dsonar.projectKey=p3-daewoon-test-user-service"
                }
            }
        }
        stage("User-Service - Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true, webhookSecretId: 'sonar-cloud-p3-daewoon-userservice-token'
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