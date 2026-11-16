
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
                    @echo off
                    set "PYTHON_CMD="

                    where py >nul 2>&1
                    if not errorlevel 1 (
                        py -3 --version
                        if not errorlevel 1 set "PYTHON_CMD=py -3"
                    )

                    if not defined PYTHON_CMD (
                        where python >nul 2>&1
                        if not errorlevel 1 (
                            python --version
                            if not errorlevel 1 set "PYTHON_CMD=python"
                        )
                    )

                    if not defined PYTHON_CMD (
                        echo ERROR: Python 3 not found.
                        echo Install Python 3 and enable Add Python to PATH.
                        echo Restart Jenkins service after installation.
                        exit /b 1
                    )

                    echo Python detected successfully
                    %PYTHON_CMD% --version

                    %PYTHON_CMD% -m pip --version
                    if errorlevel 1 (
                        %PYTHON_CMD% -m ensurepip --upgrade
                        if errorlevel 1 exit /b 1
                    )

                    %PYTHON_CMD% -m pip install --upgrade pip
                    if errorlevel 1 exit /b 1

                    %PYTHON_CMD% -m pip install -r requirements.txt
                    if errorlevel 1 exit /b 1

                    echo Dependencies installed successfully
                '''
            }
        }

        stage('Build') {
            steps {
                bat '''
                    if not exist build mkdir build
                    copy /Y index.html build\\index.html
                    if errorlevel 1 exit /b 1
                    echo Build completed successfully
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    if not exist build\\index.html exit /b 1

                    findstr /C:"Welcome to Jenkins CI/CD!" build\\index.html

                    if errorlevel 1 (
                        echo ERROR: Test failed
                        exit /b 1
                    )

                    echo Test passed successfully
                '''
            }
        }

        stage('Package') {
            steps {
                bat '''
                    powershell -NoProfile -ExecutionPolicy Bypass -Command "Compress-Archive -Path 'build\\*' -DestinationPath 'application.zip' -Force"

                    if errorlevel 1 exit /b 1
                    echo Package created successfully
                '''

                archiveArtifacts artifacts: 'application.zip',
                    fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Pipeline completed!'
        }

        failure {
            echo 'ERROR: Pipeline failed. Check the logs.'
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}
