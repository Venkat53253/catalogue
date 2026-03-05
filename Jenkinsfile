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
     /* parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password') 
    } */
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
        stage('Docker Build') {
            steps {
                script {
                     withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${acc_id}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .
                            docker push ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}

                        """
                     }
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
