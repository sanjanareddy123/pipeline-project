pipeline {
    agent any

    environment {
        EC2 = "ec2-user@3.15.31.44"
        APP_DIR = "/home/ec2-user/simple-java-app"
        S3_BUCKET = "sanjana-terraform-bucket-09843"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', 
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
                    sh "aws s3 cp target/simple-java-app-1.0.0.jar s3://${S3_BUCKET}/simple-java-app.jar"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['d16647f4-f206-4c43-ac93-10b89626c82c']) {
                    sh """
                        ssh ${EC2} "mkdir -p ${APP_DIR}"
                        scp -r target/simple-java-app-1.0.0.jar ${EC2}:${APP_DIR}/app.jar
                        ssh ${EC2} "pkill -f app.jar || true && nohup java -jar ${APP_DIR}/app.jar > app.log 2>&1 &"
                    """
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
