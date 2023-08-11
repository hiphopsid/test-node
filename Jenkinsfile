pipeline {
    agent any
 
   tools
    {
       NodeJS "nodejs-19"
    }
 stages {
      stage('checkout') {
           steps {
             
                git branch: 'main', url: 'https://github.com/hiphopsid/test-node.git'
             
          }
        }
  stage('Execute Node') {
           steps {
                sh 'rm -rf package-lock.json'
                sh 'npm cache clean --force'
                sh 'npm install' 
                sh 'npm run build'            
          }
        }
 }
}
