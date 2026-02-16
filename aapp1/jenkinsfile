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
                sh "curl 192.168.49.2:30110"
            }
        }
    }
}
