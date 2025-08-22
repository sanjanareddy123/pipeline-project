pipeline {
    agent any

    environment {
        EC2_USER = "ec2-user"
        EC2_HOST = "3.145.168.156"
        APP_DIR = "/home/ec2-user/simple-java-app"
        S3_BUCKET = "sanjana-terraform-bucket-09843"
        AWS_REGION = 'us-east-1'
        WORKSPACE_DIR = "${env.WORKSPACE}"
        SSH_KEY = "/c/Users/jsanj/downloads/vault.pem"  // path to your private key
        GIT_BASH = '"C:\\Program Files\\Git\\bin\\bash.exe"'  // Git Bash path
    }

    stages {

        stage('Checkout') {
            steps {
                bat 'git clone -b feature https://github.com/sanjanareddy123/pipeline-project .'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Test') {
            steps {
                bat 'mvn test'
            }
            post {
                always {
                    junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('Upload Artifact to S3') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    bat "\"C:\\Program Files\\Amazon\\AWSCLIV2\\aws.exe\" s3 cp \"%WORKSPACE%\\target\\simple-java-app-1.0.0.jar\" s3://%S3_BUCKET%/simple-java-app.jar --region %AWS_REGION%"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Deploy using Git Bash only
                bat """
                ${GIT_BASH} -c "scp -i ${SSH_KEY} ${WORKSPACE_DIR}/target/simple-java-app-1.0.0.jar ${EC2_USER}@${EC2_HOST}:${APP_DIR}/app.jar"
                ${GIT_BASH} -c "ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_HOST} 'mkdir -p ${APP_DIR} && pkill -f app.jar || true && nohup java -jar ${APP_DIR}/app.jar > app.log 2>&1 &'"
                """
            }
        }
    }

    post {
        success {
            echo 'Build, test, upload, and deployment succeeded!'
        }
        failure {
            echo 'Something went wrong. Check the logs.'
        }
    }
}


