node {
  stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
    nodejs('nodejs16') {
    def scannerHome = tool 'SonarScanner';
    withSonarQubeEnv() {
      sh "${scannerHome}/bin/sonar-scanner"
    }
   }
  }
  stage('Quality Gate') {
    timeout (time: 5, unit: 'MINUTES') {
       waitForQualityGate abortPipeline: true
       echo "code is good"
    }
  }
}

node('nodejs_runner') {
  stage('frontend_checkout') {
    cleanWs()
        dir ('PnLToolUI') {
        checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg:  [], \
        userRemoteConfigs: [[credentialsId: 'admingithub', url: 'git@github.com:Gemini-Solutions/PnLToolUI.git', poll: 'false']]])
        }
  }
  stage('Nodejs_Build') {
        container('nodejs-12') {
            dir ('PnLToolUI'){
              sh 'rm -rf package-lock.json'
              sh 'npm install --production'
              sh 'npm run build'
        }
       }
     }
   }

node('image_builder_trivy') {
  try {
   stage('Build_image') {
            dir ('PnLToolUI') {
              container('docker-image-builder-trivy') {
              withCredentials([usernamePassword(credentialsId: 'docker_registry', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
              sh 'docker image build -f Dockerfile -t registry-np.geminisolutions.com/pnltoolui/pnltooluibeta:1.0-$BUILD_NUMBER -t registry-np.geminisolutions.com/pnltoolui/pnltooluibeta  .'
              sh 'trivy image -f json --timeout 20m registry-np.geminisolutions.com/pnltoolui/pnltooluibeta:1.0-$BUILD_NUMBER > trivy-report.json'
	      archiveArtifacts artifacts: 'trivy-report.json', onlyIfSuccessful: true
	      withCredentials([string(credentialsId: 'Defectdojo_Token', variable: 'defectdojo_token')]) {
	      sh ''' 
		curl -X 'POST' \
		  -H 'accept: application/json' \
		  -H "Authorization: Token ${defectdojo_token}" \
		  -H 'Content-Type: multipart/form-data' \
		  -F "scan_date=$(date +%Y-%m-%d)" \
		  -F 'lead=1' \
		  -F 'minimum_severity=Info' \
		  -F 'active=true' \
		  -F 'verified=true' \
		  -F "scan_type=Trivy Scan"\
		  -F "file=@trivy-report.json;type=application/json"\
		  -F "product_name=PnLToolUI_BETA"\
		  -F "engagement_name=PnLToolUI_BETA" \
		  -F 'close_old_findings=true' \
		  -F 'push_to_jira=false' \
		  -F 'auto_create_context=true' \
		   https://defectdojo-np.geminisolutions.com//api/v2/reimport-scan/
		'''      
	}
                sh '''docker login -u $docker_user -p $docker_pass https://registry-np.geminisolutions.com'''
              sh 'docker push registry-np.geminisolutions.com/pnltoolui/pnltooluibeta:1.0-$BUILD_NUMBER'
              sh 'docker push registry-np.geminisolutions.com/pnltoolui/pnltooluibeta'
              sh 'rm -rf build/'
           }
         }
            }
      }
      stage('Deployment_stage') {
            dir ('PnLToolUI') {
              container('docker-image-builder-trivy') {
                kubeconfig(credentialsId: 'KubeConfigCred'){
                sh '/usr/local/bin/kubectl apply -f deployment-beta.yaml -n dev'
                sh '/usr/local/bin/kubectl rollout restart Deployment pnltoolui -n dev'
         }
            }
      }}
  } finally {
       sh 'echo current_image="registry.geminisolutions.com/pnltoolui/pnltooluibeta:1.0-$BUILD_NUMBER" > build.properties'
       archiveArtifacts artifacts: 'build.properties', onlyIfSuccessful: true
  }
}


