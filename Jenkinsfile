pipeline {
    agent any

    tools {
        com.cloudbees.jenkins.plugins.customtools.CustomTool 'NuGet'
    }

    environment {
    DOTNET_VERSION = '6.0.100'
    CHROME_VERSION = '137.0.7151.120'
    CHROMEDRIVER_VERSION = '137.0.7151.120'
    CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
    CHROMEDRIVER_PATH = 'C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe'
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
                bat '''
                    echo Uninstalling any existing Chrome
                    choco uninstall googlechrome -y --ignore-unfound --remove-dependencies || echo Continue if not found
                '''
                bat '''
                    echo Installing Google Chrome version %CHROME_VERSION%
                    choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums
                '''
                bat '''
                    IF EXIST "%ProgramFiles%\\Google\\Chrome\\Application\\chrome.exe" (
                        echo Chrome installation verified.
                    ) ELSE (
                        echo Chrome installation failed!
                        exit 1
                    )
                '''
            }
        }

        stage('Install matching ChromeDriver') {
            steps {
                bat '''
                    echo Downloading ChromeDriver version %CHROMEDRIVER_VERSION%
                    powershell -command "Invoke-WebRequest -Uri https://storage.googleapis.com/chrome-for-testing-public/137.0.7151.120/win64/chromedriver-win64.zip -OutFile chromedriver.zip -UseBasicParsing"
                    powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath chromedriver -Force"
                    powershell -command "Move-Item -Path .\\chromedriver\\chromedriver-win64\\chromedriver.exe -Destination '%CHROME_INSTALL_PATH%\\chromedriver.exe' -Force"
                '''
            }
        }

        stage('NuGet restore') {
            steps {
                bat 'nuget restore SeleniumIde.sln'
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
                bat 'dotnet tool install --global trx2junit'
                bat 'trx2junit SeleniumIDE/TestResults/TestResults.trx'
                junit 'SeleniumIDE/TestResults/TestResults.xml'
                archiveArtifacts artifacts: 'SeleniumIDE/TestResults/TestResults.trx', fingerprint: true
            }
        }
}
