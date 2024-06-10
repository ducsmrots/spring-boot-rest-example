pipeline {
    agent {
		label 'jdk8'
    }
    environment {
        APP_URL = 'http://localhost:8080' // Change this to your application's URL
    }
    stages {
        stage('Build and Verify') {
            steps {
                container('maven') {
                    sh 'mvn clean verify'
                    sh 'java -jar -Dspring.profiles.active=test target/spring-boot-rest-example-0.5.0.war&'
                    sleep 10
                    sh 'curl telnet://localhost:8080 -v'
                }
            }
        }
        stage('Run ZAP Scan') {
            steps {
                container('zap') {
                    script {
                        // Start ZAP in daemon mode
                        // sh "zap.sh -daemon -host 0.0.0.0 -port 8090 -config api.disablekey=true -config gui.enable=false -config callhome.disable=true"
                        // sh 'zap.sh -daemon -port 8090 -host 0.0.0.0 '
                        sh "zap-baseline.py -t http://localhost:8080"
                        // Wait for ZAP to start
                        sleep(time: 30, unit: 'SECONDS')
                        
                        // Run ZAP spider to crawl the target application
                        sh "zap-cli --zap-url http://localhost:8090 spider ${env.APP_URL}"
                        
                        // Run ZAP active scan
                        sh "zap-cli --zap-url http://localhost:8090 active-scan ${env.APP_URL}"
                        
                        // Wait for the scan to complete
                        sh 'zap-cli --zap-url http://localhost:8090 wait-for-issues -i 120'
                        
                        // Generate report
                        sh 'zap-cli --zap-url http://localhost:8090 report -o zap-report.html -f html'
                        
                        // Archive the report in Jenkins
                        archiveArtifacts artifacts: 'zap-report.html', allowEmptyArchive: true
                    }
                }
            }
        }
    }
}
