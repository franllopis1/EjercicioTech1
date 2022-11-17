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
      securityContext:
        runAsUser: 0
        privileged: true
    - name: aks
      image: acrdvpsplatformdev.azurecr.io/devops-platform-image:v0.0.5
      command:
        - sleep
      args:
        - infinity
  imagePullSecrets:
    - name: master-acr-credentials
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
        container('aks') {
          withCredentials([
              usernamePassword(
                credentialsId: 'sp-terraform-credentials',
                usernameVariable: 'AAD_SERVICE_PRINCIPAL_CLIENT_ID',
                passwordVariable: 'AAD_SERVICE_PRINCIPAL_CLIENT_SECRET'),
              string(credentialsId: 'aks-tenant', variable: 'AKS_TENANT'),
              string(credentialsId: 'aks-resource-group', variable: 'AKS_RESOURCE_GROUP')
              string(credentialsId: 'aks-name', variable: 'AKS_NAME')]) {
            sh "az login --service-principal --username ${AAD_SERVICE_PRINCIPAL_CLIENT_ID}"
            sh "az aks get-credentials --resource-group ${AKS_RESOURCE_GROUP} --name" ${AKS}
            sh 'kubelogin convert-kubeconfig -l spn'
            sh 'kubectl version'
          }
        }
      }
    }
    stage('Compile') {
      steps {
        echo '-=- Compile code -=-'
        sh './mvnw compile'
      }
    }
    stage('Unit tests') {
      steps {
        echo '-=- Execute Unit tests -=-'
        sh './mvnw test org.jacoco:jacoco-maven-plugin:report'
        //Donde estan los informes de junit
        junit 'target/surefire-reports/*.xml'
        //Donde estan los informes de jacoco
        jacoco execPattern: 'target/jacoco.exec'
      }
    }
    stage('Mutation tests') {
      steps {
        echo '-=- Execute mutation tests -=-'
        sh './mvnw org.pitest-maven:mutationCoverage'
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
        sh './mvnw package -DskipTest'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
    stage('Build & push container image') {
      steps {
        echo '-=- build & push container image -=-'
        container('podman') {
          sh "podman build -t ${CONTAINER_IMAGE_PREFIX}/${APP_NAME}:${APP_VERSION}-SNAPSHOT ."
        }
        script {
          ACR_TOKEN = 'undefined'
          container('aks') {
            withCredentials([
                UsernamePassword(
                  credentialsId: 'sp-terraform-credentials',
                  usernameVariable: 'AAD_SERVICE_PRINCIPAL_CLIENT_ID',
                  passwordVariable: 'AAD_SERVICE_PRINCIPAL_CLIENT_SECRET'),
                string(credentialsId: 'aks-tenant', variable: 'AKS_TENANT'),
                string(credentialsId: 'acr-name', variable: 'ACR_NAME')]) {
              sh "az login --service-principal --username ${AAD_SERVICE_PRINCIPAL_CLIENT_ID} --password ${AAD_SERVICE_PRINCIPAL_CLIENT_SECRET} --tenant ${AKS_TENANT}"
              ACR_TOKEN = sh(script: "az acr login -n ${ACR_NAME} --expose-token --output tsv --query accessToken",
              returnStdout: true).trim()
            }
          }
          container('podman') {
            withCredentials([string(credentialsId: 'acr-name', variable: 'ACR_NAME')]) {
              sh "podman login ${ACR_name}.azurecr.io -u 00000000-0000-0000-0000-000000000000 -P ${ACR_TOKEN}"
              sh "podman tag ${CONTAINER_IMAGE_PREFIX}/${APP_NAME}:${APP_VERSION}-SNAPSHOT ${ACR_NAME}.azurecr.io/${CONTAINER_IMAGE_PREFIX}/${APP_VERSION}-SNAPSHOT"
              sh "podman push ${ACR_NAME}.azurecr.io/${CONTAINER_IMAGE_PREFIX}/${APP_NAME}:${APP_VERSION}-SNAPSHOT"

            }
          }
        }
      }
    }
    stage('Run ephemeral test environtment') {
      steps {
        echo '-=- run ephemeral test envorontment -=-'
        container('aks') {
          withCredentials([
              usernamePassword(
                credentialsId: 'sp-terraform-credentials',
                usernameVariable: 'AAD_SERVICE_PRINCIPAL_CLIENT_ID',
                passwordVariable: 'AAD_SERVICE_PRINCIPAL_CLIENT_SECRET')]) {
            sh "kubectl run ${TEST_CONTAINER_NAME} --IMAGE=${CONTAINER_IMAGE_PREFIX}/${AP}"
            sh "kubectl expose pod ${TEST_CONTAINER_NAME} --port=${APP_LISTENING_PORT}"
            sh "kubectl expose pod ${TEST_CONTAINER_NAME} --port=${APP_JACOCO_PORT} --name"
          }
        }
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