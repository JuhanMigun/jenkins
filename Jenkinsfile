pipeline {
    agent any

    tools {
        gradle 'gradle8.2.1'
        nodejs 'nodejs'
    }

    environment {
        AWS_REGION = 'ap-northeast-2'
        ECR_REPO = '891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins'
        DOCKER_PATH = 'C:\\Program Files\\Docker\\Docker\\Resources\\bin'  // Docker 경로 설정
        PATH = "${env.PATH};${env.DOCKER_PATH}"
        DOCKER_HUB_REPO = 'kimjuhan/jenkins'  // Docker Hub 레포지토리 정보 추가
    }

    stages {
        stage('Jenkins Git Progress') {
            steps {
                git branch: 'master',
                url: 'https://github.com/JuhanMigun/jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerfileContent = """
                    FROM node:18
                    WORKDIR /root/web-1
                    COPY ./ ./
                    RUN npm install
                    ENTRYPOINT ["node","server.js"]
                    EXPOSE 8888
                    """
                    writeFile file: 'dockerfile', text: dockerfileContent
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def dockerBuildCommand = "docker build -t ${env.DOCKER_HUB_REPO}:${env.BUILD_NUMBER} ."
                    def dockerLoginCommand = "docker login -u your-dockerhub-username -p your-dockerhub-password"  // Docker Hub 로그인 명령어 추가
                    def dockerPushCommand = "docker push ${env.DOCKER_HUB_REPO}:${env.BUILD_NUMBER}"

                    if (isUnix()) {
                        sh dockerBuildCommand
                        sh dockerLoginCommand
                        sh dockerPushCommand
                    } else {
                        bat """
                        ${dockerBuildCommand}
                        ${dockerLoginCommand}
                        ${dockerPushCommand}
                        """
                    }
                }
            }
        }
    }
}