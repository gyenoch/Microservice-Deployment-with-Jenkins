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
                    credentialsId: 'app-k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://2238897EFD27563B6A2C4D501EB46649.gr7.us-east-1.eks.amazonaws.com'
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
                    credentialsId: 'app-k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://2238897EFD27563B6A2C4D501EB46649.gr7.us-east-1.eks.amazonaws.com'
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
                    credentialsId: 'elk-k8-token',
                    namespace: 'elk',
                    serverUrl: 'https://2238897EFD27563B6A2C4D501EB46649.gr7.us-east-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl apply -f ./elk"
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
                    credentialsId: 'elk-k8-token',
                    namespace: 'elk',
                    serverUrl: 'https://2238897EFD27563B6A2C4D501EB46649.gr7.us-east-1.eks.amazonaws.com'
                ]]) {
                    sh "kubectl get svc -n elk"
                }
            }
        }
    }
}
