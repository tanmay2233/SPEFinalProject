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
                script {
                    def minikubeIp = sh(script: "minikube ip", returnStdout: true).trim()
                    echo "Minikube IP: ${minikubeIp}"
                }
            }
        }

        stage('Monitor and Fetch Logs for ml-service') {
            steps {
                script {
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
                            sleep(20)
                        }
                    }
                    sh """
                    nohup kubectl logs -f ${mlServicePodName} > /var/lib/jenkins/workspace/SPE_Final/ml-service-logs.txt &
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
                        mlService2PodName = sh(script: "kubectl get pods -l app=ml-service2 -o=jsonpath='{.items[0].metadata.name}'", returnStdout: true).trim()
                        if (mlService2PodName) {
                            echo "ml-service2 pod found: ${mlService2PodName}"
                            mlService2PodReady = true
                        } else {
                            echo "Waiting for ml-service2 pod to be created..."
                            sleep(20)
                        }
                    }
                    sh """
                    nohup kubectl logs -f ${mlService2PodName} > /var/lib/jenkins/workspace/SPE_Final/ml-service2-logs.txt &
                    """
                }
            }
        }

        stage('Start FastAPI Service') {
            steps {
                script {
                    def fastApiPodName = "fastapi-service"
                    def fastApiPort = 5001
                    echo "Starting FastAPI app as a Kubernetes pod..."
                    
                    sh """
                    kubectl run ${fastApiPodName} --image=python:3.9 --restart=Never -- \
                        sh -c "pip install fastapi uvicorn requests jinja2 && uvicorn app:app --reload --host 0.0.0.0 --port ${fastApiPort}"
                    
                    echo "Exposing FastAPI service as a NodePort..."
                    kubectl expose pod ${fastApiPodName} --type=NodePort --name=fastapi-service --port=${fastApiPort}
                    """
                }
            }
        }

        stage('Monitor FastAPI Pod') {
            steps {
                script {
                    def fastApiPodReady = false
                    while (!fastApiPodReady) {
                        echo "Checking FastAPI pod..."
                        def podStatus = sh(script: "kubectl get pod fastapi-service -o=jsonpath='{.status.phase}'", returnStdout: true).trim()
                        if (podStatus == "Running") {
                            echo "FastAPI pod is in Running state."
                            fastApiPodReady = true
                        } else {
                            echo "FastAPI pod is in ${podStatus} state, waiting..."
                            sleep(10)
                        }
                    }
                }
            }
        }

        stage('Fetch FastAPI Service URL') {
            steps {
                script {
                    def minikubeIp = sh(script: "minikube ip", returnStdout: true).trim()
                    def fastApiNodePort = sh(script: "kubectl get svc fastapi-service -o=jsonpath='{.spec.ports[0].nodePort}'", returnStdout: true).trim()
                    echo "FastAPI service is accessible at: http://${minikubeIp}:${fastApiNodePort}"
                }
            }
        }

        stage('Monitor Pod Status') {
            steps {
                script {
                    def attempts = 8
                    for (int i = 1; i <= attempts; i++) {
                        sh '''
                        echo "Checking pod status... (Attempt ${i})"
                        kubectl get pods -o wide
                        '''
                        sleep(60)
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