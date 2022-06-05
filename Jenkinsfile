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