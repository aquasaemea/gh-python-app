timestamps {
	node(){
        checkout scm
        stage("Preparation"){
            sh '''
                find .
                printenv | sort
            '''
        }
        stage('Code Repository Scanned by Aqua') {
            withCredentials([
                string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'), 
                string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET'),
		string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')
            ]) {
                sh '''
                    export TRIVY_RUN_AS_PLUGIN=aqua
                    export trivyVersion=0.32.0
                    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b . v${trivyVersion}  
                    ./trivy plugin update aqua
                    ./trivy fs --debug --format template --template "@Report-Templates/aqua.tpl" -o report.html --security-checks config,vuln,secret .

                '''
            }
        }
        stage('Build Docker Image') {
            // fake build by downloading an image
	    // docker pull aquasaemea/mynodejs-app:1.0
            sh '''
	        echo the image has been built !!
            '''
        }
		
	//stage('Image Scanning by Aqua') {
        //    withCredentials([
        //        string(credentialsId: 'AQUA_REGISTRY_USER', variable: 'AQUA_REGISTRY_USER'), 
        //        string(credentialsId: 'AQUA_REGISTRY_PASSWORD', variable: 'AQUA_REGISTRY_PASSWORD'),
	//	string(credentialsId: 'AQUA_REGISTRY', variable: 'AQUA_REGISTRY'),
	//	string(credentialsId: 'AQUA_HOST', variable: 'AQUA_HOST'), 
	//	string(credentialsId: 'AQUA_SCANNER_TOKEN', variable: 'AQUA_SCANNER_TOKEN')
        //    ]) {
        //    	sh '''
	//            docker login -u "$AQUA_REGISTRY_USER" -p "$AQUA_REGISTRY_PASSWORD" $AQUA_REGISTRY
	//           docker run -e BUILD_JOB_NAME=$JOB_NAME -e BUILD_URL=$BUILD_URL -e BUILD_NUMBER=$BUILD_NUMBER --rm -v /var/run/docker.sock:/var/run/docker.sock $AQUA_REGISTRY/scanner:2022.4.46 scan --register --registry "Docker Hub" --local "aquasaemea/mynodejs-app:1.0" --host $AQUA_HOST --token $AQUA_SCANNER_TOKEN --show-negligible --html > aquascan.html
        //    	'''
	//       }
        //}
		
        stage('Manifest Generation') {
            withCredentials([
                // Replace GITLAB_CREDENTIALS_ID with the id of your gitlab credentials
                string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN'), 
                string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'), 
                string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET')
            ]) {
                // Replace ARTIFACT_PATH with the path to the root folder of your project 
                // or with the name:tag the newly built image
		            // --artifact-path "aquasaemea/mynodejs-app:1.0"
                
                sh '''
                  export BILLY_SERVER=https://prod-aqua-billy.codesec.aquasec.com
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
                        --access-token ${GITHUB_TOKEN} \
			--output sbom.json \
                        --artifact-path .
                '''
            }
        }
        post {
       	    always {
            	archiveArtifacts artifacts: '/'
       	    }
    	}
    }
}
