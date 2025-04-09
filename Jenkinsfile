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
                    dir('swe645') {
                        def mvnHome = tool 'Maven' // Use Maven installed in Jenkins
                        sh """
                            export JAVA_HOME=\$(/usr/libexec/java_home)
                            export PATH=\$JAVA_HOME/bin:\$PATH
                            ${mvnHome}/bin/mvn clean package -DskipTests
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'rutujadocker09', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASS')
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    kubectl set image deployment/swe645-deployment swe645-container=${DOCKER_IMAGE}:${DOCKER_TAG}
                    kubectl rollout restart deployment swe645-deployment
                '''
            }
        }
    }
}
