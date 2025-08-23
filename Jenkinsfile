pipeline {
    agent {label 'jenkins-agent' } // Ensure you have a Windows agent available

    tools {
        maven 'Maven3' // Make sure Jenkins has this tool installed/configured
        jdk 'Java21'   // Change to match your project's Java version
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jaabless/FakeStoreAPI_PerformanceTest.git'
            }
        }

        stage('Run JMeter Test') {
            steps {
                // Clean old reports if any exists
                bat '''
                set "WORK_DIR=%WORKSPACE:\\=/%"
                set "WORK_DIR=%WORK_DIR:/=\\%"

                if exist "%WORK_DIR%\\reports" rmdir /S /Q "%WORK_DIR%\\reports"
                if exist "%WORK_DIR%\\results.jtl" del /Q "%WORK_DIR%\\results.jtl"
                mkdir "%WORK_DIR%\\reports"
                '''

                // Run JMeter in non-GUI mode
                bat '''
                set "WORK_DIR=%WORKSPACE:\\=/%"
                set "WORK_DIR=%WORK_DIR:/=\\%"

                jmeter -n -t "%WORK_DIR%\\Magento_Performance_Testing.jmx" ^
                       -l "%WORK_DIR%\\results.jtl" ^
                       -e -o "%WORK_DIR%\\reports"
                '''
            }
        }

        stage('Verify JMeter Output') {
            steps {
                bat '''
                set "WORK_DIR=%WORKSPACE:\\=/%"
                set "WORK_DIR=%WORK_DIR:/=\\%"

                echo Results file size:
                dir "%WORK_DIR%\\results.jtl"

                echo Showing first 20 lines of results:
                powershell -Command "Get-Content '%WORK_DIR%\\results.jtl' -TotalCount 20"
                '''
            }
        }

        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'index.html',
                    reportName: 'JMeter HTML Report'
                ])
            }
        }
    }


    post {
        always {
            archiveArtifacts artifacts: 'results.jtl, reports/**', fingerprint: true
        }
    }
}