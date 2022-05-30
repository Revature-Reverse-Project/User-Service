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
                    sh "docker build -t user_service https://github.com/Revature-Reverse-Project/User-Service.git#master"
                }
            }
        }
        /*
        stage ('Docker tag and push to Github') {
            steps {
                script {
                    echo "Docker push"
                    sh "docker tag directionsapi ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/directionsapi"
                    sh "docker push ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/directionsapi"
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