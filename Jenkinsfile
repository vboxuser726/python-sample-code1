pipeline {
  agent {label 'java'}
  // triggers {
  //   pollSCM('H/2 * * * *')
  // }

   environment {
    IMAGE_NAME = "vboxuser2/py-feb26"
    Host_IP = "18.212.169.91"
    Host_Port = "30110"
    hub_cred = "dockerhub-id-cred"
    SONAR_TOKEN = credentials('sonar-token')
  }
  //  tools {
  //       sonar 'sonar-scanner'
  //   }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Check the docker cli') {
      steps {
        sh "docker --version"
        sh "docker ps -a"
      }
    }
    stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=vboxuser726_python-sample-code \
                      -Dsonar.organization=vboxuser726 \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=https://sonarcloud.io \
                      -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

      stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
      stage('Build Docker Image') {
        steps {
        sh '''
          docker build -t ${IMAGE_NAME}:$BUILD_NUMBER aapp1
        '''
      }
    }
    stage('Trivy Image Scan') {
      steps {
        sh '''
          trivy image --severity HIGH,CRITICAL --exit-code 1 ${IMAGE_NAME}:$BUILD_NUMBER
        '''
      }
    }
    stage('Push Docker Image') {
        steps {
            withCredentials([
            usernamePassword(
                credentialsId: "${hub_cred}",
                usernameVariable: 'DOCKERHUB_USER',
                passwordVariable: 'DOCKERHUB_PASS'
            )
            ]) {
            sh '''
                echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                docker push ${IMAGE_NAME}:$BUILD_NUMBER
                docker logout
            '''
            }
        }
    }

    stage('Deploy - Template and Apply') {
        agent {label 'kube'}

        steps {
            sh '''
            kubectl apply -f .
            export IMAGE_TAG=$BUILD_NUMBER
            envsubst < py-deploy.yaml | kubectl apply -f -
            kubectl rollout status deployment/py-deploy
            '''
        }
    }

    stage('Check on Kubernetes') {
        agent {label 'kube'}

      steps {
        sh '''

          kubectl get pods
          kubectl get svc
          kubectl get deployments
          
        '''
      }
    }

    stage('Application Health Check') {
      steps {
        sh '''
          sleep 10
          curl http://${Host_IP}:${Host_Port}
        '''
      }
    }
  }
}
