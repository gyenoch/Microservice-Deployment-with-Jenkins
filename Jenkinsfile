pipeline {
    agent any

    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://566BD9D26D6BEEC3E63CB61DF00C957F.gr7.us-east-1.eks.amazonaws.com/'
                ]]) {
                    sh "kubectl apply -f deployment-service.yml"
                    sleep 60
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://566BD9D26D6BEEC3E63CB61DF00C957F.gr7.us-east-1.eks.amazonaws.com/'
                ]]) {
                    sh "kubectl get svc -n webapps"
                    sleep 30
                }
            }
        }

        stage('Deploy ELK to Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'elk',
                    serverUrl: 'https://566BD9D26D6BEEC3E63CB61DF00C957F.gr7.us-east-1.eks.amazonaws.com/'
                ]]) {
                    sh "kubectl apply -f elk"
                    sleep 30
                }
            }
        }

        stage('Verify ELK Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'elk',
                    serverUrl: 'https://566BD9D26D6BEEC3E63CB61DF00C957F.gr7.us-east-1.eks.amazonaws.com/'
                ]]) {
                    sh "kubectl get svc -n elk"
                }
            }
        }
    }
}
