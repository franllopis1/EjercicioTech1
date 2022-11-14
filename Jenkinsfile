pipeline {
  agent {
    kubernetes {
      defaultContainer 'jdk'
      yaml '''
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
    - name: jdk
      image: docker.io/eclipse-temurin:18.0.2.1_1-jdk
      command:
        - sleep
      args:
        - infinity
'''
    }
  }
  stages {
    stage('Prepare environtment') {
      steps {
        echo '-=- prepare build environtment -=-'
        sh 'java -version'
      }
    }
    stage('Compile') {
      steps {
        echo '-=- Compile code -=-'
      }
    }
    stage('Unit tests') {
      steps {
        echo '-=- Execute Unit tests -=-'
      }
    }
    stage('Mutation tests') {
      steps {
        echo '-=- Execute mutation tests -=-'
      }
    }
    stage('Dependency vulnerability scans') {
      steps {
        echo '-=- Dependency vulnerability scans -=-'
      }
    }
    stage('Code inspection') {
      steps {
        echo '-=- Code inspection -=-'
      }
    }
    stage('Package') {
      steps {
        echo '-=- Package application-=-'
      }
    }
    stage('Build & push container image') {
      steps {
        echo '-=- build & push container image -=-'
      }
    }
    stage('Run ephemeral test environtment') {
      steps {
        echo '-=- run ephemeral test envorontment -=-'
      }
    }
    stage('Performance test') {
      steps {
        echo '-=- Execute performance test -=-'
      }
    }
    stage('Promote container image') {
      steps {
        echo '-=- promote container image -=-'
      }
    }
  }
  post {
    always {
        echo '-=- clearing up resources -=-'
    }
  }
}