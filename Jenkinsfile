
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        PYTHON = 'C:\\Users\\Dell\\AppData\\Local\\Programs\\Python\\Python310\\python.exe'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                echo 'Checking required project files...'

                bat '''
                    @echo off

                    if not exist requirements.txt (
                        echo ERROR: requirements.txt not found!
                        exit /b 1
                    )

                    if not exist index.html (
                        echo ERROR: index.html not found!
                        exit /b 1
                    )

                    echo All required files are present.
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'

                bat '''
                    @echo off

                    echo Checking Python installation...
                    "%PYTHON%" --version
                    if errorlevel 1 exit /b 1

                    echo Checking pip...
                    "%PYTHON%" -m pip --version

                    if errorlevel 1 (
                        "%PYTHON%" -m ensurepip --upgrade
                        if errorlevel 1 exit /b 1
                    )

                    echo Installing requirements...
                    "%PYTHON%" -m pip install -r requirements.txt
                    if errorlevel 1 exit /b 1

                    echo Dependencies installed successfully!
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'

                bat '''
                    @echo off

                    if exist build rmdir /s /q build

                    mkdir build

                    copy index.html build\\index.html

                    echo Build completed successfully!
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing the application...'

                bat '''
                    @echo off

                    if not exist build\\index.html (
                        echo ERROR: Build output not found!
                        exit /b 1
                    )

                    findstr /C:"Welcome to Jenkins CI/CD!" build\\index.html

                    if errorlevel 1 (
                        echo ERROR: Expected welcome text not found!
                        exit /b 1
                    )

                    echo All tests passed successfully!
                '''
            }
        }

        stage('Package') {
            steps {
                echo 'Creating application ZIP package...'

                powershell '''
                    if (Test-Path "application.zip") {
                        Remove-Item "application.zip" -Force
                    }

                    Compress-Archive `
                        -Path "build\\*" `
                        -DestinationPath "application.zip" `
                        -Force

                    Write-Host "Application packaged successfully!"
                '''

                archiveArtifacts artifacts: 'application.zip', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Jenkins CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'FAILURE: Jenkins CI/CD pipeline failed. Check the Console Output.'
        }

        always {
            echo 'Jenkins pipeline execution finished.'
        }
    }
}
