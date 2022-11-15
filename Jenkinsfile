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
    - name: podman
      image: quay.io/containers/podman:v4.2.0
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
        sh './mvnw --version'
        container('podman') {
          sh 'podman --version'
        }
      }
    }
  }
}