pipeline {
    agent any
    environment {
        PROJECT_ID = 'project-id'
        CLUSTER_NAME = 'cluster-name'
        CLUSTER_LOCATION = 'northamerica-northeast2'
        REGISTRY_LOCATION = 'northamerica-northeast2'
        REPOSITORY = 'repository-name'
        CREDENTIALS_ID = 'credentials-id'
    }
    stages {
        stage('Quality Gate') {
            steps {
                echo "Quality Gate"
            }
        }
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
                    sh "docker tag user-service ghcr.io/jamty1/user-service"
                    sh "docker push ghcr.io/jamty1/user-service"
                }
            }
        }
        /*
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