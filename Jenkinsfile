pipeline {
    agent any
    tools {
        maven "my-maven"
        dockerTool "my-docker"
    }
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
        // stage ('Docker Build') {
        //     steps {
        //         script {
        //             sh "docker build -t user-service ."
        //         }
        //     }
        // }
        // stage ('Docker tag and push to Google Artifact Registry') {
        //     steps {
        //         // script {
        //         //     sh "docker tag user-service ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
        //         //     sh "docker push ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
        //         // }
        //     }
        // }
        podTemplate(yaml: '''
            apiVersion: v1
            kind: Pod
            spec:
            containers:
            - name: maven
                image: maven:3.8.1-jdk-8
                command:
                - sleep
                args:
                - 99d
            - name: kaniko
                image: gcr.io/kaniko-project/executor:debug
                command:
                - sleep
                args:
                - 9999999
                volumeMounts:
                - name: kaniko-secret
                mountPath: /secret
                env: 
                - name: kaniko-secret
                mountPath: /secret
            restartPolicy: Never
            volumes:
            - name: kaniko-secret
                secret:
                    secretName: kaniko-secret
        ''') {
            node(POD_LABEL) {
                stage('Get a Maven project') {
                git url: 'https://github.com/Revature-Reverse-Project/User-Service', branch: 'master'
                container('maven') {
                    stage('Build a Maven project') {
                    sh '''
                    echo pwd
                    '''
                    }
                }
                }

                stage('Build Java Image') {
                container('kaniko') {
                    stage('Build a Go project') {
                    sh '''
                        /kaniko/executor --context `pwd` --destination gcr.io/reverse-devops-sre/user-service:1.0
                    '''
                    }
                }
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
