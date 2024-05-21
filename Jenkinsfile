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
                script {
                    if (isUnix()) {
                        sh 'aws ecr-public get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
                        sh 'docker build -t $ECR_REPO:$BUILD_NUMBER .'
                        sh 'docker push $ECR_REPO:$BUILD_NUMBER'
                    } else {
                        bat 'aws ecr-public get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_REPO%'
                        bat 'docker build -t %ECR_REPO%:%BUILD_NUMBER% .'
                        bat 'docker push %ECR_REPO%:%BUILD_NUMBER%'
                    }
                }
            }
        }

        stage('Update Manifest ArgoCD') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'git config --global user.email "gong2401@gmail.com"'
                        sh 'git config --global user.name "JuhanMigun"'
                    } else {
                        bat 'git config --global user.email "gong2401@gmail.com"'
                        bat 'git config --global user.name "JuhanMigun"'
                    }

                    withCredentials([usernamePassword(credentialsId: 'github-sjh', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        if (isUnix()) {
                            sh '''
                            git checkout argocd
                            git pull origin argocd
                            sed -E -i "s~(image: $ECR_REPO:)(backend-v[0-9]+.[0-9]+|[0-9]+)~image: $ECR_REPO:$BUILD_NUMBER~g" argo/infra.yaml
                            git add argo/infra.yaml
                            git commit -m "Update image in infra.yaml"
                            git push origin argocd
                            '''
                        } else {
                            bat '''
                            git checkout argocd
                            git pull origin argocd
                            powershell -Command "(Get-Content argo/infra.yaml) -replace '(image: $ECR_REPO:)(backend-v[0-9]+.[0-9]+|[0-9]+)', 'image: $ECR_REPO:$BUILD_NUMBER' | Set-Content argo/infra.yaml"
                            git add argo/infra.yaml
                            git commit -m "Update image in infra.yaml"
                            git push origin argocd
                            '''
                        }
                    }
                }
            }
        }
    }
}
