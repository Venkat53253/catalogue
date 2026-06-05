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
        stage('Trigger Deploy') {
            when{
                expression { params.deploy }
            }
            steps {
                script {
                    build job: 'catalogue-cd',
                    parameters: [
                        string(name: 'appVersion', value: "${appVersion}"),
                        string(name: 'deploy_to', value: 'dev')
                    ],
                    propagate: false,  // even SG fails VPC will not be effected
                    wait: false // VPC will not wait for SG pipeline completion
                }
            }
        }
        stage ('test') {
            steps {
                script {
                    echo 'building'
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
        stage('Sonar Scan') {
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
        }
        stage('Trigger Deploy') {
            when {
                expression { params.deploy == true }   // ✅ valid param
            }
            steps {
                script {
                    build job: 'catalogue-cd',
                          parameters: [
                              string(name: 'appVersion', value: "${env.appVersion}"),
                              string(name: 'deploy_to', value: 'dev')
                          ],
                          propagate: false,
                          wait: false
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
