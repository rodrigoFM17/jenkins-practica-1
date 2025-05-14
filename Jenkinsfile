pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
        EC2_USER = 'ubuntu'
        EC2_MAIN_IP = '52.72.70.22'
        EC2_DEV_IP = "184.73.177.137"
        EC2_QA_IP = "34.230.217.105"
        REMOTE_PATH = '/home/ubuntu/jenkins-practica-1'
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'rm -rf node_modules'
                sh 'npm ci'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def branch = env.GIT_BRANCH
                    echo branch
                    def ip = ""
                    if(branch == "main"){
                        ip = env.EC2_MAIN_IP
                    } else if(branch == "dev"){
                        ip = env.EC2_DEV_IP
                    } else if(branch == "qa") {
                        ip = env.EC2_QA_IP
                    } else {
                        error("no hay un servidor para esta rama ${env.BRACH_NAME}")
                    }

                    sh """
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip '
                        cd $REMOTE_PATH &&
                        git pull origin ${branch} &&   
                        npm ci &&
                        pm2 restart health-api || pm2 start server.js --name health-api
                    '
                    """
                }    
            }
        }
    }
}
