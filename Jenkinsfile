pipeline {
    agent any

    environment {
        EC2_NAME = "MyEC2" // Name configured in Jenkins Publish Over SSH global settings
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
                    echo "Uploading artifact to S3..."
                    bat '"C:\\Program Files\\Amazon\\AWSCLIV2\\aws.exe" s3 cp "%WORKSPACE%\\target\\simple-java-app-1.0.0.jar" s3://%S3_BUCKET%/simple-java-app.jar --region %AWS_REGION%'
                }
            }
        }

        stage('Deploy to EC2 via Publish Over SSH') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: "${EC2_NAME}", // Matches the global config name
                            transfers: [
                                sshTransfer(
                                    // Option 1: copy from Jenkins workspace
                                    sourceFiles: 'target/simple-java-app-1.0.0.jar',
                                    removePrefix: 'target',
                                    remoteDirectory: '', // leave blank, folder created in execCommand
                                    // Option 2: copy from S3 (comment Option 1 if using S3)
                                    // execCommand below handles both directory creation & starting app
                                    execCommand: '''
                                        mkdir -p /home/ec2-user/simple-java-app &&
                                        # Option 1: already copied from Jenkins workspace
                                        # Option 2: Uncomment next line if copying from S3
                                        # aws s3 cp s3://sanjana-terraform-bucket-09843/simple-java-app.jar /home/ec2-user/simple-java-app/simple-java-app.jar &&
                                        pkill -f app.jar || true &&
                                        nohup java -jar /home/ec2-user/simple-java-app/simple-java-app-1.0.0.jar > app.log 2>&1 &
                                    '''
                                )
                            ],
                            usePromotionTimestamp: false
                        )
                    ]
                )
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
