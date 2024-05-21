pipeline {
    agent any

    tools {
        gradle 'gradle8.2.1'
        nodejs 'nodejs'
    }

    stages {
        stage('Jenkins Git Progress') {
            steps {
                git branch: 'main',
                url: 'https://github.com/JuhanMigun/jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'echo "FROM node:18" > dockerfile' 
                sh 'echo "WORKDIR /root/web-1" >> dockerfile' 
                sh 'echo "COPY ./ ./" >> dockerfile'
                sh 'echo "RUN npm install" >> dockerfile'
                sh '''
                echo 'ENTRYPOINT ["node","server.js"]' >> dockerfile
                '''
                sh 'echo "EXPOSE 8888" >> dockerfile'
            }
        }

        stage('Image push to ECR') {
            steps {
                sh 'aws ecr-public get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com'
                sh 'docker build -t 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins:$BUILD_NUMBER .'
                sh 'docker push 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins:$BUILD_NUMBER'
            }
        }
        
        stage('Update Manifest ArgoCD') {
            steps {
                sh 'git config --global user.email "gong2401@gmail.com"'
                sh 'git config --global user.name "JuhanMigun"'

                withCredentials([gitUsernamePassword(credentialsId: 'github-sjh', gitToolName: 'Default')]) {
                    sh 'git checkout argocd'
                    sh 'git pull origin argocd'
                    
                    sh 'sed -E -i "s~(image: 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins:)(backend-v[0-9]+.[0-9]+|[0-9]+)~image: 891377163278.dkr.ecr.ap-northeast-2.amazonaws.com/jenkins:$BUILD_NUMBER~g" argo/infra.yaml'
                    sh 'git add argo/infra.yaml'
                    sh 'git commit -m "Update image in infra.yaml"'
                    sh 'git push origin argocd'
                }
            }
        }
    }
}