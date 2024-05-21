pipeline {
    agent any

    tools {
        gradle 'gradle8.2.1'
        nodejs 'nodejs'
    }

    environment {
        AWS_REGION = 'ap-northeast-2'
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
                    writeFile file: 'dockerfile', text: dockerfileContent
                }
            }
        }

        stage('Image push to ECR') {
            steps {
                sh 'aws ecr-public get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com'
                sh 'docker build -t 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins:$BUILD_NUMBER .'
                sh 'docker push 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins:$BUILD_NUMBER'
            }
        }
    }
}