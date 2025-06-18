pipeline {
    agent any

    environment {
        DOTNET_VERSION = "6.0.100"
        CHROME_VERSION = "137.0.7151.104"
        CHROMEDRIVER_VERSION = "137.0.7151.104"
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

        stage('Install Chrome and ChromeDriver fallback') {
            steps {
                bat '''
                    echo Uninstalling existing Chrome if any...
                    choco uninstall googlechrome -y || echo Chrome not installed

                    echo Installing Chrome version %CHROME_VERSION%
                    choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums

                    echo Downloading matching ChromeDriver %CHROMEDRIVER_VERSION%
                    powershell -Command "Invoke-WebRequest -Uri https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/%CHROMEDRIVER_VERSION%/win64/chromedriver-win64.zip -OutFile chromedriver.zip"
                    powershell -Command "Expand-Archive -Path chromedriver.zip -DestinationPath . -Force"
                    powershell -Command "Move-Item -Path .\\chromedriver-win64\\chromedriver.exe -Destination 'C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe' -Force"
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