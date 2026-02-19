pipeline {
    agent { label 'java' }
triggers {
    pollSCM('H/2 * * * *')
}
    stages {
        stage('Git Clone') {
            steps {
                echo "Cloning GitHub repo..."
                checkout scm
            }
        }
        stage('Check the docker cli') {
      steps {
        sh "docker --version"
        sh "docker ps -a"
      }
    }
        stage('Build Docker Image') {
             steps {
                 sh '''
          docker build -t vboxuser2/py-feb26:$BUILD_NUMBER aapp1
        '''
      }
    }
        stage('Push Docker Image') {
            steps {
            withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-id-cred',
                usernameVariable: 'DOCKERHUB_USER',
                passwordVariable: 'DOCKERHUB_PASS'
            )
            ]) {
            sh '''
                echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                docker push vboxuser2/py-feb26:$BUILD_NUMBER
                docker logout
            '''
            }
        }
    }
        stage('Run Kubernetes Manifests') {
            steps {
                sh "kubectl get pods"
		        sh "kubectl get rs"
		        sh "kubectl get deployment"
                echo "Applying Kubernetes YAML files..."
                sh '''
                  kubectl apply -f .
                '''
            }
        }
        stage('verification') {
            steps {
                sh "sleep 20"
                sh "curl 192.168.49.2:30110"
            }
        }
    }
}
