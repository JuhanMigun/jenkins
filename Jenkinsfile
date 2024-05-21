pipeline {
    agent any

    tools {
        gradle 'gradle8.2.1'
        nodejs 'nodejs'
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-northeast-2' // AWS_DEFAULT_REGION 대신 AWS_REGION 사용
        ECR_REPO = '891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins'
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
                    writeFile file: 'Dockerfile', text: dockerfileContent // 'Dockerfile' 대신 'dockerfile'을 사용하시면 안 됩니다.
                }
            }
        }

        stage('Image push to ECR') {
            steps {
                script {
                    sh 'aws ecr-public get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REPO'
                    sh "docker build -t $ECR_REPO:latest ." // $BUILD_NUMBER 대신 latest 태그 사용
                    sh "docker push $ECR_REPO:latest" // $BUILD_NUMBER 대신 latest 태그 사용
                }
            }
        }
    }
}
