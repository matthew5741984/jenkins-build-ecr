pipeline {
    agent any

    environment {
        // GitHub credentials
        GIT_REPOSITORY = 
        CREDENTIALS_ID = 'my-gitlab-repo-creds'

        // Laravel Environment
        APP_DEBUG = 'true'
        DB_HOST = 
        DB_DATABASE = 'blog'
        DB_USERNAME = credentials('DB_USERNAME')
        DB_PASSWORD = credentials('DB_PASSWORD')
        
        // Laravel AWS Environment
        LARAVEL_AWS_IAM_KEY = credentials('LARAVEL_AWS_IAM_KEY')
        LARAVEL_AWS_IAM_SECRET = credentials('LARAVEL_AWS_IAM_SECRET')
        AWS_DEFAULT_REGION = 'eu-west-2'
        AWS_BUCKET = 
        AWS_URL = 

        // ECR login credentials
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        REPOSITORY_NAME = 
        REPOSITORY_ADDRES = 
        REPOSITORY_REGION = 

    }
    stages {
        stage('git-clone') {
            steps {
                dir('php/cdam-news') {
                    git branch: 'master', credentialsId: "${CREDENTIALS_ID}", url: "${GIT_REPOSITORY}"
                }
            }
        }
        stage('webapp-init') {
            steps {
                dir('php/cdam-news') {
                    sh 'cp .env.example .env'
                    sh "sed -i -e 's/APP_DEBUG=false/APP_DEBUG=${APP_DEBUG}/g' .env"
                    sh "sed -i -e 's/DB_HOST=/DB_HOST=${DB_HOST}/g' .env"
                    sh "sed -i -e 's/DB_DATABASE=/DB_DATABASE=${DB_DATABASE}/g' .env"
                    sh "sed -i -e 's/DB_USERNAME=/DB_USERNAME=${DB_USERNAME}/g' .env"
                    sh "sed -i -e 's/DB_PASSWORD=/DB_PASSWORD=${DB_PASSWORD}/g' .env"
                    sh "sed -i -e 's/LARAVEL_AWS_IAM_KEY=/LARAVEL_AWS_IAM_KEY=${LARAVEL_AWS_IAM_KEY}/g' .env"
                    sh "sed -i -e 's%LARAVEL_AWS_IAM_SECRET=%LARAVEL_AWS_IAM_SECRET=${LARAVEL_AWS_IAM_SECRET}%g' .env"
                    sh "sed -i -e 's/AWS_DEFAULT_REGION=/AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}/g' .env"
                    sh "sed -i -e 's/AWS_BUCKET=/AWS_BUCKET=${AWS_BUCKET}/g' .env"
                    sh "sed -i -e '0,/AWS_URL=/s%AWS_URL=%AWS_URL=${AWS_URL}%g' .env" 
                }  
            }
        }      
        stage('docker-build') {
            steps {
                sh "docker build -t $REPOSITORY_NAME:$BUILD_ID php"
                sh "aws ecr get-login-password --region $REPOSITORY_REGION | docker login --username AWS --password-stdin $REPOSITORY_ADDRES"
                sh "docker tag $REPOSITORY_NAME:$BUILD_ID $REPOSITORY_ADDRES/$REPOSITORY_NAME:$BUILD_ID"
                sh "docker push $REPOSITORY_ADDRES/$REPOSITORY_NAME:$BUILD_ID"
                sh "rm -rf php/cdam-news"
                sh "rm -rf php/cdam-news@tmp"
            }
        }
    }

    post {
        always {
            echo "Clean up docker build image"
            sh "docker rmi $REPOSITORY_NAME:$BUILD_ID"
            sh "docker rmi $REPOSITORY_ADDRES/$REPOSITORY_NAME:$BUILD_ID"
        }
            
        failure {
                sh "rm -rf php/cdam-news"
                sh "rm -rf php/cdam-news@tmp"
        }
    }
}
