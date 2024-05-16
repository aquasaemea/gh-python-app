pipeline {
  agent any
  stages {
    stage('Aqua Code Scanning') {
      agent {
        docker {
          image 'aquasec/aqua-scanner'    
        }
      }
      steps {
        withCredentials([
          string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'),
          string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET'),
          usernamePassword(credentialsId: 'PROVIDER_CREDENTIALS', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')
        ]){
          sh '''
            export TRIVY_RUN_AS_PLUGIN=aqua
            trivy fs --scanners config,vuln,secret --sast --reachability .

            # To customize which severities to scan for, add the following flag: --severity UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
            # To enable SAST scanning, add: --sast
            # To enable reachability scanning, add: --reachability
            # To enable npm/dotnet/gradle non-lock file scanning, add: --package-json / --dotnet-proj / --gradle
            # For http/https proxy configuration add env vars: HTTP_PROXY/HTTPS_PROXY, CA-CRET (path to CA certificate)
          '''
        }
      }
    }
    stage('Build Docker Image') {
      steps {
            sh '''
	            echo the image has been built !!
            '''
      }      
    }
    /*stage('Aqua Image Scanning ') {
      steps {
        withCredentials([
          string(credentialsId: 'AQUA_REGISTRY_USER', variable: 'AQUA_REGISTRY_USER'), 
          string(credentialsId: 'AQUA_REGISTRY_PASSWORD', variable: 'AQUA_REGISTRY_PASSWORD'),
		      string(credentialsId: 'AQUA_HOST', variable: 'AQUA_HOST'), 
		      string(credentialsId: 'AQUA_SCANNER_TOKEN', variable: 'AQUA_SCANNER_TOKEN')
          ]){
            	sh '''
	              docker login -u "$AQUA_REGISTRY_USER" -p "$AQUA_REGISTRY_PASSWORD" registry.aquasec.com
	              docker run -e BUILD_JOB_NAME=$JOB_NAME -e BUILD_URL=$BUILD_URL -e BUILD_NUMBER=$BUILD_NUMBER --rm -v /var/run/docker.sock:/var/run/docker.sock registry.aquasec.com/scanner:2404.30.6 scan --register --registry "Docker Hub" --local "aquasaemea/mynodejs-app:1.0" --host $AQUA_HOST --token $AQUA_SCANNER_TOKEN --show-negligible --html > aquascan.html
            	'''
	        }
      }
    }*/
    
    stage('Aqua SBOM Next Generation') {
      steps {
        withCredentials([
          // Replace BITBUCKET_TOKEN with the id of your bitbucket token
          string(credentialsId: 'BITBUCKET_TOKEN', variable: 'BITBUCKET_TOKEN'),
          string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'), 
          string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET')
          ]) {
                // Replace ARTIFACT_PATH with the path to the root folder of your project 
                // or with the name:tag the newly built image     
                sh '''
                  export BILLY_SERVER=https://billy.codesec.aquasec.com
            	    curl -sLo install.sh download.codesec.aquasec.com/billy/install.sh
            	    curl -sLo install.sh.checksum https://github.com/argonsecurity/releases/releases/latest/download/install.sh.checksum
                  if ! cat install.sh.checksum | sha256sum ; then
                      echo "install.sh checksum failed"
                      exit 1
                  fi
                  BINDIR="." sh install.sh
                  rm install.sh install.sh.checksum
                    ./billy generate -v \
                        --aqua-key ${AQUA_KEY} \
                        --aqua-secret ${AQUA_SECRET} \
                        --access-token  ${BITBUCKET_TOKEN}  \
                        --artifact-path .

                   # The docker image name:tag of the newly built image
                   # --artifact-path "my-image-name:my-image-tag" 
                   # OR the path to the root folder of your project. I.e my-repo/my-app 
                   # --artifact-path "ARTIFACT_PATH"
                '''
            }
        }
    }  
  }
}
