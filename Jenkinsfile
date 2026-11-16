pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out'
            }
        }

        stage('Verify Files') {
            steps {
                bat '''
                    if not exist requirements.txt (
                        echo ERROR: requirements.txt not found
                        exit /b 1
                    )
                    if not exist index.html (
                        echo ERROR: index.html not found
                        exit /b 1
                    )
                    echo Required files found
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    python --version
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Build') {
            steps {
                bat '''
                    if not exist build mkdir build
                    copy /Y index.html build\\index.html
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    if not exist build\\index.html exit /b 1
                    findstr /C:"Welcome to Jenkins CI/CD!" build\\index.html
                    if errorlevel 1 (
                        echo ERROR: Expected welcome text was not found
                        exit /b 1
                    )
                    echo Test passed
                '''
            }
        }

        stage('Package') {
            steps {
                bat '''
                    powershell -NoProfile -ExecutionPolicy Bypass -Command "Compress-Archive -Path 'build\\*' -DestinationPath 'application.zip' -Force"
                    if errorlevel 1 exit /b 1
                '''
                archiveArtifacts artifacts: 'application.zip', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Pipeline completed!'
        }
        failure {
            echo 'ERROR: Pipeline failed. Check the stage logs above.'
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}
