pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"  // Disable host key checking for Ansible
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/tanmay2233/SPEFinalProject'
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh 'minikube version' // Print Minikube version
                sh 'ansible-playbook -i localhost, --connection=local deploy_services.yml'
            }
        }

        stage('Monitor Pod Status') {
            steps {
                script {
                    def attempts = 10  // Number of attempts (20 minutes / 2 minutes per check)
                    for (int i = 1; i <= attempts; i++) {
                        sh '''
                        echo "Checking pod status... (Attempt ${i})"
                        kubectl get pods -o wide
                        '''
                        sleep 120 // Wait for 2 minutes
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Final verification..."
                kubectl get pods -o wide
                kubectl get services
                echo "ml-service should be available at http://192.168.49.2:<PORT>"
                echo "ml-service2 should be available at http://192.168.49.2:<PORT>"
                '''
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
            echo 'Deployment failed. Check logs for details.'
        }
    }
}
