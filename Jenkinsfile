// Rutuja More

pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'rutujadocker09/swe645'
        DOCKER_TAG = 'latest'
        // KUBECONFIG_PATH = 'C:\\Program Files\\Jenkins\\kubeconfig'
        KUBECONFIG_PATH = '~/.kube/config'
    }
    stages {
         stage('Clone Repository') {
            steps {
                // Clone the repository
                git branch: 'main',
                    credentialsId: 'rutujamore01', 
                    url: 'https://github.com/rutujamore01/swe645.git'
            }
        }
        stage('Build JAR File') {
            steps {
                script {
                    dir('swe645') {
                        def mvnHome = tool 'Maven' // Use Maven installed in Jenkins
                        bat "${mvnHome}/bin/mvn clean package -DskipTests"
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(credentialsId: 'rutujadocker09', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASS')
                    ]) {
                        bat "echo logging into Docker Hub..."
                        bat "echo | set /p=\"%DOCKER_PASS%\" | docker login --username %DOCKER_USERNAME% --password-stdin"
                        bat "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                // Using the Kubeconfig to update the Kubernetes deployment
                // bat "kubectl set image deployment/swe645-deployment swe645-container=${DOCKER_IMAGE}:${DOCKER_TAG} --kubeconfig=\"C:\\Program Files\\Jenkins\\kubeconfig\""
                bat "kubectl set image deployment/swe645-deployment swe645-container=${DOCKER_IMAGE}:${DOCKER_TAG} --kubeconfig=\"${KUBECONFIG_PATH}\""
                // Restart the pods to reflect the new changes
                // bat "kubectl rollout restart deployment swe645-deployment --kubeconfig=\"C:\\Program Files\\Jenkins\\kubeconfig\""
                bat "kubectl rollout restart deployment swe645-deployment --kubeconfig=\"${KUBECONFIG_PATH}\""
            }
        }
    }
}