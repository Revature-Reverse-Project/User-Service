pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "building master"
        withMaven{
          sh 'mvn package -DskipTests'
        }
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

    stage('Test') {
      steps {
        echo "Testing branch"
        withMaven{
          sh 'mvn test'
        }
      }
    }
    stage('Docker Image') {
      steps {
        sh 'echo "docker build"'
        script{
          docker.build "user-service"
        }
      }
    }
    stage('Deploy') {
      steps {
        sh 'echo "Hello, World"'
      }
    }
  }
}