pipeline {
    agent any

    environment {
        PYTHON_PATH = 'C:\\ProgramData\\Anaconda3\\python.exe'
        PATH = "C:\\ProgramData\\Anaconda3;C:\\ProgramData\\Anaconda3\\Library\\bin;C:\\ProgramData\\Anaconda3\\Scripts;${env.PATH}"
        PYTHONUTF8 = '1'
        PYTHONIOENCODING = 'utf-8'
    }

    stages {
        stage('Verify Python Path') {
            steps {
                bat '''
                    @echo off
                    "%PYTHON_PATH%" --version
                    if errorlevel 1 exit /b 1
                    "%PYTHON_PATH%" -c "import ssl; print(ssl.OPENSSL_VERSION)"
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Fix pip') {
            steps {
                bat '''
                    @echo off
                    "%PYTHON_PATH%" -m pip --version
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    @echo off
                    "%PYTHON_PATH%" -m pip install -r requirements.txt
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Lint') {
            steps {
                bat '''
                    @echo off
                    "%PYTHON_PATH%" -m flake8 app.py tests/
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    @echo off
                    "%PYTHON_PATH%" -m pytest --cov=app tests/ --cov-report=html
                    if errorlevel 1 exit /b 1
                '''
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

                stage('Build') {
            steps {
                bat '''
                    @echo off
                    "%PYTHON_PATH%" -m venv .build-venv
                    if errorlevel 1 exit /b 1

                    ".build-venv\\Scripts\\python.exe" -m pip install -r requirements.txt pyinstaller
                    if errorlevel 1 exit /b 1

                    ".build-venv\\Scripts\\python.exe" -m PyInstaller --clean --noconfirm --onefile app.py
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                withEnv(['JENKINS_NODE_COOKIE=flaskapp']) {
                    bat '''
                        @echo off
                        start "" "%PYTHON_PATH%" app.py > flaskapp.log 2>&1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD pipeline failed!'
        }
    }
}
