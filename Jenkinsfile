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
                if exist "%WORKSPACE%\\reports" rmdir /S /Q "%WORKSPACE%\\reports"
                if exist "%WORKSPACE%\\results.jtl" del /Q "%WORKSPACE%\\results.jtl"
                mkdir "%WORKSPACE%\\reports"
                '''

                // Run JMeter in non-GUI mode
                bat '''
                jmeter -n -t "%WORKSPACE%\\Magento_Performance_Testing.jmx" ^
                       -l "%WORKSPACE%\\results.jtl" ^
                       -e -o "%WORKSPACE%\\reports"
                '''
            }
        }

        stage('Verify JMeter Output') {
            steps {
                bat '''
                echo Results file size:
                dir "%WORKSPACE%\\results.jtl"
                echo Showing first 20 lines of results:
                for /f "tokens=* skip=0 delims=" %%a in ('type "%WORKSPACE%\\results.jtl" ^| more +0') do (
                    echo %%a
                    set /a count+=1
                    if !count! geq 20 exit /b 0
                )
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