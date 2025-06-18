pipeline {
    agent any

    environment {
        DOTNET_VERSION = "6.0.100"
        CHROME_VERSION = "137.0.7151.104"
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/MarinaKoceva/SeleniumIde.git'
            }
        }

        stage('Install .NET SDK and Runtime') {
            steps {
                bat "echo Installing .NET SDK ${DOTNET_VERSION}"
                bat "choco install dotnet-sdk --version=${DOTNET_VERSION} -y"

                bat "echo Installing .NET Desktop Runtime ${DOTNET_VERSION}"
                bat "choco install dotnet-desktopruntime --version=6.0.0 -y"
            }
        }

        stage('Ensure Chrome version') {
            steps {
                bat "echo Checking if Chrome is installed"
                bat '''
                    choco list --localonly | findstr googlechrome >nul
                    IF %ERRORLEVEL%==0 (
                        echo Chrome is installed. Proceeding with uninstall...
                        choco uninstall googlechrome -y
                    ) ELSE (
                        echo Chrome not installed. Skipping uninstall.
                    )
                '''
                bat "echo Installing Google Chrome version ${env.CHROME_VERSION}"
                bat "choco install googlechrome --version=${env.CHROME_VERSION} -y --allow-downgrade --ignore-checksums"
            }
        }

        stage('Install matching ChromeDriver') {
            steps {
                bat '''
                    echo Downloading ChromeDriver version 137.0.7151.104
                    powershell -command "Invoke-WebRequest -Uri https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/137.0.7151.104/win64/chromedriver-win64.zip -OutFile chromedriver.zip -UseBasicParsing"
                    powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath . -Force"
                    powershell -command "Move-Item -Path .\\chromedriver-win64\\chromedriver.exe -Destination 'C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe' -Force"
                '''
            }
        }

        stage('Restore dependencies') {
            steps {
                bat 'dotnet restore SeleniumIde.sln'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build SeleniumIde.sln --configuration Release'
            }
        }

        stage('Run tests') {
            steps {
                bat 'dotnet test SeleniumIde.sln --logger "trx;LogFileName=TestResults.trx"'
            }
        }
    }

    post {
        always {
            junit '**/TestResults/*.trx'
            archiveArtifacts artifacts: '**/TestResults/*.trx', fingerprint: true
        }
    }
}