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

        stage('Print Minikube IP') {
            steps {
                sh '''
                echo "Fetching Minikube IP..."
                MINIKUBE_IP=$(minikube ip)
                echo "Minikube IP: ${MINIKUBE_IP}"
                '''
            }
        }

        stage('Monitor and Fetch Logs for ml-service') {
            steps {
                script {
                    // Monitor until the ml-service pod is ready
                    def mlServicePodName = ""
                    def mlServicePodReady = false
                    while (!mlServicePodReady) {
                        echo "Checking ml-service pod..."
                        mlServicePodName = sh(script: "kubectl get pods -l app=ml-service -o=jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
                        if (mlServicePodName) {
                            echo "ml-service pod found: ${mlServicePodName}"
                            mlServicePodReady = true
                        } else {
                            echo "Waiting for ml-service pod to be created..."
                            sleep 20  // Wait for 20 seconds before retrying
                        }
                    }

                    // Once the ml-service pod is created, fetch logs using nohup
                    sh """
                    nohup kubectl logs -f ${mlServicePodName} > /var/lib/jenkins/workspace/SPE_Final/ml-service-logs.txt &
                    """
                }
            }
        }

        stage('Monitor and Fetch Logs for ml-service2') {
            steps {
                script {
                    // Monitor until the ml-service2 pod is ready
                    def mlService2PodName = ""
                    def mlService2PodReady = false
                    while (!mlService2PodReady) {
                        echo "Checking ml-service2 pod..."
                        mlService2PodName = sh(script: "kubectl get pods -l app=ml-service2 -o=jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
                        if (mlService2PodName) {
                            echo "ml-service2 pod found: ${mlService2PodName}"
                            mlService2PodReady = true
                        } else {
                            echo "Waiting for ml-service2 pod to be created..."
                            sleep 20  // Wait for 20 seconds before retrying
                        }
                    }

                    // Once the ml-service2 pod is created, fetch logs using nohup
                    sh """
                    nohup kubectl logs -f ${mlService2PodName} > /var/lib/jenkins/workspace/SPE_Finalml-service2-logs.txt &
                    """
                }
            }
        }

        stage('Monitor Pod Status') {
            steps {
                script {
                    def attempts = 15  
                    for (int i = 1; i <= attempts; i++) {
                        sh '''
                        echo "Checking pod status... (Attempt ${i})"
                        kubectl get pods -o wide
                        '''
                        sleep 60 // Wait for 1 minute
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
