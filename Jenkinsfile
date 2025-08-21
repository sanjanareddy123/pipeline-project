pipeline {
    agent any

    environment {
        EC2 = "ec2-user@3.15.31.44"
        APP_DIR = "/home/ec2-user/simple-java-app"
        S3_BUCKET = "sanjana-terraform-bucket-09843"
        AWS_REGION = 'us-east-1'
        WORKSPACE_DIR = "${env.WORKSPACE}"
    }

    
    stages {

        stage('Checkout') {
            steps {
                git branch: 'feature', 
                    url: 'https://github.com/sanjanareddy123/pipeline-project'
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
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    // Print the workspace path (debugging)
                    bat 'echo Workspace: %WORKSPACE%'

                    // Check the file exists
                    bat 'dir "%WORKSPACE%\\target"'

                    // Upload the artifact
                    bat "\"C:\\Program Files\\Amazon\\AWSCLI\\bin\\aws.exe\" s3 cp \"%WORKSPACE%\\target\\simple-java-app-1.0.0.jar\" s3://%S3_BUCKET%/simple-java-app.jar"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['d16647f4-f206-4c43-ac93-10b89626c82c']) {
                    // Copy artifact to EC2
                    bat "scp \"%WORKSPACE%\\target\\simple-java-app-1.0.0.jar\" %EC2%:%APP_DIR%/app.jar"

                    // Restart app on EC2
                    bat "ssh %EC2% \"mkdir -p %APP_DIR% && pkill -f app.jar || true && nohup java -jar %APP_DIR%/app.jar > app.log 2>&1 &\""
                }
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
