pipeline {
    agent any
    environment {
        PROJECT_ID = 'project-id'
        CLUSTER_NAME = 'cluster-name'
        CLUSTER_LOCATION = 'northamerica-northeast2'
        CREDENTIALS_ID = 'credentials-id'
        SCANNER_HOME = tool 'SonarQubeScanner'
        ORGANIZATION = "revature-reverse-project"
        PROJECT_NAME = "User-Service"
    }
    stages {
        stage('Quality Gate') {
            steps {
                echo "Quality Gate"
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        /*
        stage('Unit Tests') {
            steps {
                echo "Unit Tests"
                sh "mvn test"
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                    echo "Docker Build"
                    sh "docker build -t user-service ."
                }
            }
        }
        stage ('Docker tag and push to Github') {
            steps {
                script {
                    echo "Docker push"
                    sh "docker tag user-service ghcr.io/${GITHUB_USER}/user-service"
                    sh "docker push ghcr.io/${GITHUB_USER}/user-service"
                }
            }
        }
        stage ('Deploy to GKE') {
            steps {
                echo "Deploying to GKE"
                step([$class: 'KubernetesEngineBuilder',
                    projectId: env.PROJECT_ID,
                    clusterName: env.CLUSTER_NAME,
                    location: env.CLUSTER_LOCATION,
                    manifestPattern: 'deployment',
                    credentialsId: env.CREDENTIALS_ID,
                    verifyDeployments: true])
            }
        }
        */
    }
}