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
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password') 
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
