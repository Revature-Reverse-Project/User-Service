pipeline {
    agent any
    stages {
        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar \
                        -Dsonar.organization=$ORGANIZATION \
                        -Dsonar.java.binaries=target'
                }
            }
        }
       stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                    sh "docker build -t user-service ."
                }
            }
        }
        stage ('Docker tag and push to Google Artifact Registry') {
            steps {
                script {
                    sh "docker tag user-service ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
                    sh "docker push ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
                }
            }
        }
        stage ('Deploy to GKE') {
            steps {
                sh "sed -i 's|image: user-service|image: ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service|g' Kubernetes/user-service.yaml"
                step([$class: 'KubernetesEngineBuilder',
                    projectId: env.PROJECT_ID,
                    clusterName: env.CLUSTER_NAME,
                    location: env.CLUSTER_LOCATION,
                    manifestPattern: 'Kubernetes',
                    credentialsId: env.CREDENTIALS_ID,
                    verifyDeployments: true])
            }
        }
    }
}
