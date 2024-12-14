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

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods -o wide
                '''
            }
        }

        stage('Print Service Details') {
            steps {
                script {
                    def services = sh(
                        script: 'kubectl get services -o jsonpath="{range .items[*]}{.metadata.name} {.spec.clusterIP} {.spec.ports[*].port}\\n{end}"',
                        returnStdout: true
                    ).trim()

                    echo "Service details (Name, ClusterIP, Port):"
                    echo services

                    def urls = sh(
                        script: 'minikube service list',
                        returnStdout: true
                    ).trim()

                    echo "Access your services at these URLs (NodePort or LoadBalancer):"
                    echo urls
                }
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
