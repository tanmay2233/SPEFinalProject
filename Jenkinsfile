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
                    def mlServicePodName = ""
                    def mlServicePodReady = false

                    while (!mlServicePodReady) {
                        echo "Checking ml-service pod..."
                        // Fetch the pod name
                        mlServicePodName = sh(script: "kubectl get pods -l app=ml-service -o=jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
                        if (mlServicePodName) {
                            echo "ml-service pod found: ${mlServicePodName}"
                            // Check if the pod is in the 'Running' state
                            def podStatus = sh(script: "kubectl get pod ${mlServicePodName} -o=jsonpath='{.status.phase}'", returnStdout: true).trim()
                            if (podStatus == "Running") {
                                echo "ml-service pod is in Running state"
                                mlServicePodReady = true
                            } else {
                                echo "ml-service pod is in ${podStatus} state, waiting..."
                                sleep(20) // Wait 20 seconds before rechecking
                            }
                        } else {
                            echo "ml-service pod not yet created, waiting..."
                            sleep(20) // Wait 20 seconds before retrying
                        }
                    }

                    // Fetch logs using nohup once the pod is Running
                    sh """
                    nohup kubectl logs -f ${mlServicePodName} > ml-service-logs.txt &
                    """
                }
            }
        }

        stage('Monitor and Fetch Logs for ml-service2') {
            steps {
                script {
                    def mlService2PodName = ""
                    def mlService2PodReady = false

                    while (!mlService2PodReady) {
                        echo "Checking ml-service2 pod..."
                        // Fetch the pod name
                        mlService2PodName = sh(script: "kubectl get pods -l app=ml-service2 -o=jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
                        if (mlService2PodName) {
                            echo "ml-service2 pod found: ${mlService2PodName}"
                            // Check if the pod is in the 'Running' state
                            def podStatus = sh(script: "kubectl get pod ${mlService2PodName} -o=jsonpath='{.status.phase}'", returnStdout: true).trim()
                            if (podStatus == "Running") {
                                echo "ml-service2 pod is in Running state"
                                mlService2PodReady = true
                            } else {
                                echo "ml-service2 pod is in ${podStatus} state, waiting..."
                                sleep(20) // Wait 20 seconds before rechecking
                            }
                        } else {
                            echo "ml-service2 pod not yet created, waiting..."
                            sleep(20) // Wait 20 seconds before retrying
                        }
                    }

                    // Fetch logs using nohup once the pod is Running
                    sh """
                    nohup kubectl logs -f ${mlService2PodName} > ml-service2-logs.txt &
                    """
                }
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
