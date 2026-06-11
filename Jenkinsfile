@Library('jenkins-shared-lib') _

def configMap = [
    project : "roboshop",
    component : "catalogue"
]

if( ! env.BRANCH_NAME.equalsIgnoreCase('main') ){
  nodejsEKSPipeline(configMap) // by default it will call, call function inside this pipeline
}
else{
    echo "Please proceed with PROD process"
}