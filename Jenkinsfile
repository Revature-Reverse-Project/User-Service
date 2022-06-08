pipeline {
    agent {
        kubernetes {
          defaultContainer 'jnlp'
          yaml """
          apiVersion: v1
          kind: Pod
          metadata:
          labels:
            component: ci
          spec:
            containers:
            - name: jnlp
              image: ikenoxamos/jenkins-slave:latest
              workingDir: /home/jenkins
              env:
              - name: DOCKER_HOST
                value: tcp://localhost:2375
              resources:
                requests:
                  memory: "500Mi"
                  cpu: "0.3"
                limits:
                  memory: "800Mi"
                  cpu: "0.5"
            - name: dind-daemon
              image: docker:18-dind
              workingDir: /var/lib/docker
              securityContext:
                privileged: true
              volumeMounts:
              - name: docker-storage
                mountPath: /var/lib/docker
              resources:
                requests:
                  memory: "300Mi"
                  cpu: "0.3"
                limits:
                  memory: "500Mi"
                  cpu: "0.5"
            - name: kubectl
              image: jshimko/kube-tools-aws:latest
              command:
              - cat
              tty: true
            volumes:
            - name: docker-storage
              emptyDir: {}
          """
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
        // stage ('Docker tag and push to Google Artifact Registry') {
        //     steps {
        //         script {
        //             sh "docker tag user-service ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
        //             sh "docker push ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service"
        //         }
        //     }
        // }
        stage ('Deploy to GKE') {
            steps {
                container('kubectl') {
                    withKubeConfig([credentialsId: 'env.CREDENTIALS_ID']) {
                        sh 'kubectl set image deployment/user-service user-service=${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service'
                    }
                }
                // sh "sed -i 's|image: user-service|image: ${REGISTRY_LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/user-service|g' Kubernetes/user-service.yaml"
                // step([$class: 'KubernetesEngineBuilder',
                //     projectId: env.PROJECT_ID,
                //     clusterName: env.CLUSTER_NAME,
                //     location: env.CLUSTER_LOCATION,
                //     manifestPattern: 'Kubernetes',
                //     credentialsId: env.CREDENTIALS_ID,
                //     verifyDeployments: true])
            }
        }
    }
}
