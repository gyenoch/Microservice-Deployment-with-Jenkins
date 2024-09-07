pipeline {
    agent any

    stages {
        stage('Deploy Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://fa041c7db9fd359c452d1fabc391cd3f.gr7.us-east-1.eks.amazonaws.com/'
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
                    serverUrl: 'https://fa041c7db9fd359c452d1fabc391cd3f.gr7.us-east-1.eks.amazonaws.com/'
                ]]) {
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}

