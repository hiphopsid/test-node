// pipeline {
//     agent any

//     tools {
//         nodejs "nodejs-19"
//     }

//     stages {
//         stage('checkout') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/hiphopsid/test-node.git'
//             }
//         }
//         stage('Execute Node') {
//             steps {
//                 sh 'rm -rf package-lock.json'
//                 sh 'npm cache clean --force'
//                 sh 'npm install'
//                 sh 'npm run build'
//             }
//         }
//     }
// }
pipeline {
    agent {
        docker {
            image 'node:19-slim
        }
    }
     environment {
            CI = 'true'
        }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

    }
}
