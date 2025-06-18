pipeline {
    agent any

    environment {
        CHROME_VERSION = '127.0.6533.73'
        CHROMEDRIVER_VERSION = '127.0.6533.72'
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROMEDRIVER_PATH = 'C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/MarinaKoceva/SeleniumIde.git'
            }
        }

        stage('Set up .NET Core') {
            steps {
                bat '''
                echo Installing .NET SDK 6.0
                choco install dotnet-sdk -y --version=6.0.100
                '''
            }
        }

        stage('Install .NET Runtime') {
            steps {
                bat '''
                echo Installing .NET Desktop Runtime 6.0
                choco install dotnet-desktopruntime --version=6.0.0 -y
                '''
            }
        }

        stage('Uninstall Current Chrome') {
            steps {
                bat '''
                echo Checking if Chrome is installed
                choco list --localonly | findstr googlechrome > nul
                IF %ERRORLEVEL% EQU 0 (
                    echo Chrome is installed. Proceeding with uninstall...
                    choco uninstall googlechrome -y
                ) ELSE (
                    echo Chrome not installed. Skipping uninstall.
                )
                exit 0
                '''
            }
        }

        stage('Install Specific Version of Chrome') {
            steps {
                bat '''
                echo Installing Google Chrome version %CHROME_VERSION%
                choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums
                '''
            }
        }

        stage('Download and Install ChromeDriver') {
            steps {
                bat '''
                echo Downloading ChromeDriver version %CHROMEDRIVER_VERSION%
                powershell -command "Invoke-WebRequest -Uri https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/137.0.7151.104/win64/chromedriver-win64.zip -OutFile chromedriver.zip -UseBasicParsing"
                powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath ."
                powershell -command "Move-Item -Path .\\chromedriver-win64\\chromedriver.exe -Destination '%CHROME_INSTALL_PATH%\\chromedriver.exe' -Force"
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
            archiveArtifacts artifacts: '**/TestResults/**/*.trx', allowEmptyArchive: true
            junit '**/TestResults/**/*.trx'
        }
    }
}
