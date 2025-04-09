// Rutuja More

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'rutujadocker09/swe645'
        DOCKER_TAG = 'latest'
        KUBECONFIG_PATH = "${HOME}/.kube/config"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'rutujamore01',
                    url: 'https://github.com/rutujamore01/swe645.git'
            }
        }

        stage('Build JAR File') {
            steps {
                script {
                    dir('SWE-645') {
                        def mvnHome = tool 'Maven'
                        sh """
                            export JAVA_HOME=/opt/homebrew/opt/openjdk@23
                            export PATH=\$JAVA_HOME/bin:\$PATH
                            echo "Java Version:"
                            java -version
                            ${mvnHome}/bin/mvn clean package -DskipTests
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    export PATH=/opt/homebrew/bin:/usr/local/bin:\$PATH
                    echo "Docker Version:"
                    docker --version
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'rutujadocker09',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        export PATH=/opt/homebrew/bin:/usr/local/bin:\$PATH
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    export PATH=/opt/homebrew/bin:/usr/local/bin:\$PATH
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    kubectl set image deployment/swe645-deployment swe645-container=${DOCKER_IMAGE}:${DOCKER_TAG}
                    kubectl rollout restart deployment swe645-deployment
                """
            }
        }
    }
}
