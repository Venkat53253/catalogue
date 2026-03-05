pipeline {
    agent {
        label 'agent1'
    }
  /*   enviornment {
        course = 'jenkins'
    } */
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
        stage ('build') {
            steps {
                script {
                    sh """
                       echo "hello build"
                       sleep 10
                       env
                    """
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
            deleteder()    
        }
        success {
            echo 'sucess'
        }
        failure {
            echo 'fail'
        }
    }
}