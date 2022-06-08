pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
            name: kaniko
            spec:
            containers:
              - name: kaniko
                image: gcr.io/kaniko-project/executor:latest
                args:
                  - "--dockerfile=Dockerfile"
                  - "--context=git://github.com/Revature-Reverse-Project/User-Service"
                  - "--destination=gcr.io/reverse-devops-sre/user-service:1.0"
                volumeMounts:
                  - name: kaniko-secret
                    mountPath: /secret
                env:
                  - name: GOOGLE_APPLICATION_CREDENTIALS
                    value: /secret/kaniko-secret.json
            restartPolicy: Never
            volumes:
              - name: kaniko-secret
                secret:
                    secretName: kaniko-secret
                
        '''
        }
    }
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
        stage ('Docker tag and push to Google Artifact Registry') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor --context git://github.com/Revature-Reverse-Project/User-Service --destination gcr.io/reverse-devops-sre/user-service:1.0 --dockerfile Dockerfile
                    '''
                }
            }
                // script {
                //     sh "docker tag user-service ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
                //     sh "docker push ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
                // }
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
