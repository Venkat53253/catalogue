pipeline {
    agent {
        label 'agent1'
    }
     environment {
        appVersion = ''
        region = 'us-east-1'
        acc_id = '622072308398'
        project = 'roboshop'
        component = 'catalogue'
    }
    options {
              timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
     parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value') 
    } 
    // Build
    stages {
        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }
        stage('Install Dependances') {
            steps {
                script {
                    sh """
                        npm install

                    """
                }
            }
        }
        stage ('UNIT TEST') {
            steps {
                script {
                    sh """
                        echo "test"
                    """
                }
            }
        }
        /* stage('Sonar Scan') {
            environment {
                scannerHome = tool 'sonar-8.2'
            }
            steps {
                script {
                   // Sonar Server envrionment
                   withSonarQubeEnv(installationName: 'sonar-8.2') {
                         sh "${scannerHome}/bin/sonar-scanner"
                   }
                }
            }
        } */
        // Enable webhook in sonarqube server and wait for results
        /* stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true }
            }
        } */ 
        stage('Check Dependabot Alerts') {
            environment { 
                GITHUB_TOKEN = credentials('github-token')
            }
            steps {
                script {
                    // Fetch alerts from GitHub
                    def response = sh(
                        script: """
                            curl -s -H "Accept: application/vnd.github+json" \
                                 -H "Authorization: token ${GITHUB_TOKEN}" \
                                 https://api.github.com/repos/Venkat53253/catalogue/dependabot/alerts
                        """,
                        returnStdout: true
                    ).trim()

                    // Parse JSON
                    def json = readJSON text: response

                    // Filter alerts by severity
                    def criticalOrHigh = json.findAll { alert ->
                        def severity = alert?.security_advisory?.severity?.toLowerCase()
                        def state = alert?.state?.toLowerCase()
                        return (state == "open" && (severity == "critical" || severity == "high"))
                    }

                    if (criticalOrHigh.size() > 0) {
                        error "❌ Found ${criticalOrHigh.size()} HIGH/CRITICAL Dependabot alerts. Failing pipeline!"
                    } else {
                        echo "✅ No HIGH/CRITICAL Dependabot alerts found."
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }
        stage('Check Scan Results') {
         steps {
         script {
            withAWS(credentials: 'aws-cred', region: 'us-east-1') {

                sh """
                    aws ecr start-image-scan \
                    --repository-name ${PROJECT}/${COMPONENT} \
                    --image-id imageTag=${appVersion} \
                    --region ${REGION} || true

                    sleep 60
                """

                def findings = sh(
                    script: """
                        aws ecr describe-image-scan-findings \
                        --repository-name ${PROJECT}/${COMPONENT} \
                        --image-id imageTag=${appVersion} \
                        --region ${REGION} \
                        --output json
                    """,
                    returnStdout: true
                ).trim()

                def json = readJSON text: findings

                def highCritical = json.imageScanFindings.findings.findAll {
                    it.severity == "HIGH" || it.severity == "CRITICAL"
                }

                if (highCritical.size() > 0) {
                    error("Build failed due to vulnerabilities")
                } else {
                    echo "✅ No HIGH/CRITICAL vulnerabilities found."
                }
            }
        }
    }
}

    post {
        always {
            echo 'not completed'
            deleteDir()    
        }
        success {
            echo 'sucess'
        }
        failure {
            echo 'fail'
        }
    }
    
}
