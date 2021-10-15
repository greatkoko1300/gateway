def account_id="979050235289"
def region_code="ap-southeast-2"
def image_registry="${account_id}"+".dkr.ecr."+"${region_code}"+".amazonaws.com/"
def image_name="${image_registry}"+"${env.JOB_NAME}"
def email_recipients="acmexii@naver.com"

notifyMail("STARTED", "${email_recipients}")

podTemplate(label: '12-mall',
	containers: [
        containerTemplate(name: 'git', image: 'alpine/git', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'maven', image: 'maven:alpine', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat')
  ],
  volumes: [ 
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'), 
    persistentVolumeClaim(mountPath: '/root/.m2/repository', claimName: 'maven-repo', readOnly: false)
  ]
) {
    node('12-mall') { 
        def appImage
	env.image_name="${image_name}"     // This Variable can be referenced in GitOps K8s Manifests 
        
        try {
	        stage('Checkout'){
	            container('git'){
	                checkout scm
	            }
	        }
	         stage('Compile & Test'){
	            container('maven'){
	                script {
	                    sh 'mvn package'
	                }
	            }
	        }
	        stage('Build'){
	            container('docker'){
	                script {
	                    appImage = docker.build("${image_name}")
	                }
	            }
	        }
	        stage('Push'){		
	            container('docker'){
	                script {		
	                    docker.withRegistry("https://"+"${image_registry}", "ecr:ap-southeast-2:ecr-cred"){
	                        appImage.push("${env.BUILD_NUMBER}")
	                        appImage.push("latest")
	                    }
	                }
	            }
	        }	      
	        stage('Deploy'){
	            container('git'){
			        kubernetesDeploy(kubeconfigId: 'kube-cred', 	// REQUIRED
		                 configs: '**/kubernetes/*.yaml',   		// REQUIRED
		                 enableConfigSubstitution: true
		            )
	            }
	         }	         
	        notifyMail("${currentBuild.currentResult}", "${email_recipients}")
	    } catch (e) {
	    	currentBuild.result = "FAILED"
	    	notifyMail("${currentBuild.currentResult}", "${email_recipients}")
	    } 
    }    
}

def notifyMail(STATUS, RECIPIENTS) {
	emailext body: STATUS+" : " +  "${env.JOB_NAME} [${env.BUILD_NUMBER}] (${env.BUILD_URL})",
	subject: STATUS + " : " + "${env.JOB_NAME} [${env.BUILD_NUMBER}]",
	to: RECIPIENTS
}
