pipeline {
    agent any

    // environment {
    //     ANSIBLE_HOST_KEY_CHECKING = "False" // Disable host key checking
    // }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/tanmay2233/SPEFinalProject'
            }
        }

        stage('Set Up Minikube Context') {
            steps {
                sh '''
                minikube start
                kubectl config use-context minikube
                '''
            }
        }

        stage('Deploy Services with Ansible') {
            steps {
                sh '''
                ansible-playbook -i localhost, --connection=local deploy_services.yml
                '''
            }
        }

        stage('Verify Services') {
            steps {
                sh 'kubectl get pods -o wide'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
