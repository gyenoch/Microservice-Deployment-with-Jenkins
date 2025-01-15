pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        NVD_API_KEY = credentials('nvd-api-key')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/currencyservice']],
                    userRemoteConfigs: [[url: 'https://github.com/gyenoch/Microservice-Deployment-with-Jenkins.git']]
                ])
            }
        }

        stage('Sonarqube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=currencyservice \
                    -Dsonar.projectKey=currencyservice \
                    -Dsonar.exclusions=**/*.java
                    '''
                }
            }
        }

        // stage("quality gate"){
        //    steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
        //         }
        //     } 
        // }

        stage('OWASP Dependency-Check') {
            steps {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey ${NVD_API_KEY}', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy File Scan') {
            steps {
                    sh 'trivy fs . >> trivyfs.txt'
                    script {
                        def scanResults = readFile('trivyfs.txt')
                        if (scanResults.contains('CRITICAL')) {
                            echo "Warning: Critical vulnerabilities found in frontend file scan!"
                        }
                    }
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t gyenoch/currencyservice:${BUILD_NUMBER} ."
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image gyenoch/currencyservice:${BUILD_NUMBER} >> trivyimage.txt'
                script {
                    def scanResults = readFile('trivyimage.txt')
                    // Log the scan results without throwing an error
                    echo "Currencyservice scan results:\n${scanResults}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push gyenoch/currencyservice:${BUILD_NUMBER} "
                    }
                }
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "Microservice-Deployment-with-Jenkins"
                GIT_USER_NAME = "gyenoch"
                DEPLOYMENT_FILE = "deployment-service.yml"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_APP')]) {
                    sh '''
                        set -e  # Exit immediately if a command exits with a non-zero status
                        
                        echo "Configuring Git user details..."
                        git config user.email "www.gyenoch@gmail.com"
                        git config user.name "gyenoch"

                        echo "Checking out the main branch..."
                        git checkout main

                        echo "Pulling the latest changes..."
                        git pull origin main

                        echo "Extracting current image tag from ${DEPLOYMENT_FILE}..."
                        imageTag=$(grep 'image:\\s*gyenoch/currencyservice:' ${DEPLOYMENT_FILE} | sed -E 's/.*gyenoch\\/currencyservice:([a-zA-Z0-9._-]+)/\\1/')
                        if [ -z "$imageTag" ]; then
                            echo "Error: Current image tag not found in ${DEPLOYMENT_FILE}!"
                            exit 1
                        fi
                        echo "Current image tag: $imageTag"

                        echo "Updating image tag to BUILD_NUMBER=${BUILD_NUMBER}..."
                        sed -i "s|gyenoch/currencyservice:$imageTag|gyenoch/currencyservice:$BUILD_NUMBER|" ${DEPLOYMENT_FILE}

                        echo "Staging changes..."
                        git add ${DEPLOYMENT_FILE}

                        echo "Committing the updated file..."
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit."

                        echo "Pushing changes to the main branch..."
                        git push https://${GITHUB_APP}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main || {
                            echo "Push failed! Pulling latest changes and retrying..."
                            git pull origin main
                            git push https://${GITHUB_APP}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        }
                    '''
                }
            }
        }
    }
}

// pipeline {
//     agent any

//     stages {
//         stage('Build & Tag Docker Image') {
//             steps {
//                 script {
//                     withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
//                         sh "docker build -t gyenoch/currencyservice:latest ."
//                     }
//                 }
//             }
//         }
        
//         stage('Push Docker Image') {
//             steps {
//                 script {
//                     withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
//                         sh "docker push gyenoch/currencyservice:latest "
//                     }
//                 }
//             }
//         }
//     }
// }
