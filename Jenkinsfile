
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out'
            }
        }

        stage('Build') {
            steps {
                echo 'Building application'
                bat '''
                    if not exist build mkdir build
                    copy /Y index.html build\\index.html
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing application'
                bat '''
                    if not exist build\\index.html exit /b 1
                    findstr /C:"Welcome to Jenkins CI/CD!" build\\index.html
                    if errorlevel 1 exit /b 1
                    echo Test passed
                '''
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application'
                bat '''
                    powershell -NoProfile -Command "Compress-Archive -Path build\\* -DestinationPath application.zip -Force"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Pipeline completed!'
        }
        failure {
            echo 'ERROR: Pipeline failed!'
        }
    }
}
