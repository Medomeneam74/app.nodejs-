pipeline { 
    agent any

    environment {
        REPO_URL = "https://github.com/Medomeneam74/app.nodejs-.git"
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: "${env.GIT_BRANCH}", credentialsId: 'github', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    
                    def cleanBranch = env.GIT_BRANCH ? env.GIT_BRANCH.replaceAll('origin/', '').toLowerCase() : 'main'
                    env.CLEAN_APP_NAME = "new-app-${cleanBranch}"
                    
                    sh """
                        docker build -t ${env.CLEAN_APP_NAME}:${env.BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                            docker tag ${env.CLEAN_APP_NAME}:${env.BUILD_NUMBER} ${DOCKER_USERNAME}/${env.CLEAN_APP_NAME}:${env.BUILD_NUMBER}
                            docker push ${DOCKER_USERNAME}/${env.CLEAN_APP_NAME}:${env.BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                
                    sh """
                        docker rm -f "${env.CLEAN_APP_NAME}-${env.BUILD_NUMBER}" || true  
                        docker run -p 500${env.BUILD_NUMBER}:5000 --name "${env.CLEAN_APP_NAME}-${env.BUILD_NUMBER}" -d ${env.CLEAN_APP_NAME}:${env.BUILD_NUMBER}
                        docker ps
                    """
                }
            }
        }
    }
}