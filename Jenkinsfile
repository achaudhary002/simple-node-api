pipeline {
  agent any

  environment {
    DOCKERHUB_CRED = credentials('dockerhub')
    GITHUB_CRED = credentials('github')
    APP_NAME = "node-cicd-demo"
    DOCKER_IMAGE = "achaudhary002/${APP_NAME}"
  }

  stages {

    stage('Checkout Code'){
      steps {
        git branch: 'main', url: 'https://github.com/achaudhary002/node-cicd-demo.git', credentialsID: "${GITHUB_CRED}"
      }
    }
    stage('Install & Test'){
      steps {
        steps {
          sh '''
            npm install
            npm test
        '''
          }
        }
    }
    stage('Build Docker Image'){
      steps {
        sh " docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
      }
    }
    stage('Push to Dockerhub'){
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]){
          sh '''
            echo $PASS | docker login -u $USER --password-stdin
            docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
            docker push ${DOCKER_IMAGE}:latest
        '''
        }
       }
   }
    stage('Deploy to the Kubernetes'){
      steps {
        sh '''
          kubectl apply -f k8s/Deployment.yaml
          kubectl apply -f k8s/Service.yaml
        '''
        }
       }
   }
    post {
      success {
        echo "Built and Deploy success: ${BUILD_NUMBER}"
      }
      failure {
        echo " XX Build Failed:  ${BUILD_NUMBER}"
      }
   }
}
