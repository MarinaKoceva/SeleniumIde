pipeline {
    agent any

    environment {
        CHROME_DRIVER_VERSION = ''
        CHROME_VERSION = ''
    }

    stage('Checkout code') {
    steps {
        git branch: 'main', url: 'https://github.com/MarinaKoceva/SeleniumIde.git'
    }
}

        stage('Install .NET SDK and Runtime') {
            steps {
                bat 'choco install dotnet-sdk -y --version=6.0.100'
                bat 'choco install dotnet-desktopruntime --version=6.0.0 -y'
            }
        }

        stage('Ensure Chrome version') {
            steps {
                bat '''
                powershell -Command "(Get-ItemProperty -Path \"HKLM:\\Software\\Google\\Chrome\\BLBeacon\").version" > chrome_version.txt
                '''
                script {
                    def ver = readFile('chrome_version.txt').trim()
                    env.CHROME_VERSION = ver
                    env.CHROME_DRIVER_VERSION = ver.replaceAll(/\.[^.]+$/, '') // Trim last dot group
                }
            }
        }

        stage('Install matching ChromeDriver') {
            steps {
                bat '''
                echo Downloading ChromeDriver version %CHROME_DRIVER_VERSION%
                powershell -Command "Invoke-WebRequest -Uri https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/%CHROME_DRIVER_VERSION%/win64/chromedriver-win64.zip -OutFile chromedriver.zip -UseBasicParsing"
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
            junit '**/TestResults.trx'
            archiveArtifacts artifacts: '**/TestResults.trx', allowEmptyArchive: true
        }
    }
}