pipeline {
    agent any
    stages {
        stage('Code Analysis') {
            steps {
                echo "Code Analysis"
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                        #!/bin/bash -xe
                        if [[ "$CI_BRANCH_NAME" == $BRANCH ]]; then
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar
                            -Dsonar.organization=$ORGANIZATION
                            -Dsonar.java.binaries=target
                            -Dsonar.branch.name=$BRANCH
                        fi
                        '''
                }
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
        stage ('Docker tag and push to Google Artifact Registry') {
            steps {
                script {
                    echo "Docker push"
                    sh "docker tag user-service ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
                    sh "docker push ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
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
    }
}