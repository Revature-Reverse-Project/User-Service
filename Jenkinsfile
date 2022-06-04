pipeline {
  agent any
  stages {
    stage('Build') {
      when{
        branch 'master'
      }
      steps {
        echo "building master"
        withMaven{
          sh 'mvn package -DskipTests'
        }
      }
    }
    stage('Test') {
      when{
        not {branch 'master'}
      }
      steps {
        echo "Testing branch"
        withMaven{
          sh 'mvn test'
        }
      }
    }
    stage('Docker Image') {
      steps {
        sh 'echo "Hello, World"'
      }
    }
    stage('Deploy') {
      steps {
        sh 'echo "Hello, World"'
      }
    }
  }
}