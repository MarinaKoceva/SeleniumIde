pipeline {
    agent any

    environment {
        DOTNET_VERSION = '6.0.100'
        DOTNET_RUNTIME_VERSION = '6.0.0'
        CHROME_VERSION = '127.0.6533.73' // fallback if auto fails
    }

    stages {
        stage('Checkout code') {
            steps {
                git url: 'https://github.com/MarinaKoceva/SeleniumIde.git'
            }
        }

        stage('Set up .NET Core') {
            steps {
                bat """
                echo Installing .NET SDK ${DOTNET_VERSION}
                choco install dotnet-sdk --version=${DOTNET_VERSION} -y
                """
            }
        }

        stage('Install .NET Runtime') {
            steps {
                bat """
                echo Installing .NET Desktop Runtime ${DOTNET_RUNTIME_VERSION}
                choco install dotnet-desktopruntime --version=${DOTNET_RUNTIME_VERSION} -y
                """
            }
        }

        stage('Install Specific Chrome if not present') {
            steps {
                bat """
                echo Checking if Chrome is installed
                choco list --localonly | findstr googlechrome >nul
                if %ERRORLEVEL% EQU 0 (
                    echo Chrome is already installed.
                ) else (
                    echo Installing Google Chrome ${CHROME_VERSION}
                    choco install googlechrome --version=${CHROME_VERSION} -y --allow-downgrade --ignore-checksums
                )
                """
            }
        }

        stage('Download and Install Matching ChromeDriver') {
            steps {
                bat """
                echo Detecting installed Chrome version...

                for /f "tokens=3" %%v in ('reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\App Paths\chrome.exe" /ve ^| find "\\"') do set ChromePath=%%v
                for /f "tokens=3 delims= " %%v in ('"%%ChromePath%%" --version') do set ChromeVer=%%v
                echo Installed Chrome version: %ChromeVer%

                set "MajorVer="
                for /f "tokens=1 delims=." %%a in ("%ChromeVer%") do set MajorVer=%%a
                echo Detected Chrome major version: %MajorVer%

                echo Cleaning up old driver...
                powershell -command "Remove-Item -Recurse -Force chromedriver.zip, chromedriver-win64 -ErrorAction SilentlyContinue"

                echo Downloading ChromeDriver for version %ChromeVer%...
                set ChromeDriverURL=https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/%ChromeVer%/win64/chromedriver-win64.zip
                powershell -command "Invoke-WebRequest -Uri %ChromeDriverURL% -OutFile chromedriver.zip -UseBasicParsing"

                echo Extracting driver...
                powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath . -Force"

                echo Moving to Chrome folder...
                powershell -command "Move-Item -Path .\\chromedriver-win64\\chromedriver.exe -Destination 'C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe' -Force"
                """
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
            archiveArtifacts artifacts: '**/TestResults/*.trx', allowEmptyArchive: true
            junit '**/TestResults/*.trx'
        }
    }
}
