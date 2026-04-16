pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kothaavinash/student-results:latest"  
        KUBECONFIG = 'C:\\Users\\HP\\.kube\\config'            //check path
    } 

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Avinash2005-s/practise.git'   //git repo
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker push %DOCKER_IMAGE%'
                }
            }
        }

        stage('Check Kubernetes Connection') {
            steps {
                bat 'kubectl cluster-info'   // ✅ Debug step
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
            }
        }
    }

    post {
        success { 
            echo 'CI/CD Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check logs.'
        }
    }
}
