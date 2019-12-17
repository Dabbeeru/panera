final def tagName = env.TAG_NAME
final def branchName = env.BRANCH_NAME

properties([
  // only keep 25 builds to prevent disk usage from growing out of control
  buildDiscarder(
    logRotator(
      artifactDaysToKeepStr: '', 
      artifactNumToKeepStr: '', 
      daysToKeepStr: '', 
      numToKeepStr: '25',
    ),
  ),
])

pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-slave
    user: technology-platform-235213
spec:
  containers:
  - name: jenkins-slave
    image: gcr.io/technology-platform-235213/sharedservices-jenkins-slave-jnlp:0.0.5
    command:
    - cat
    tty: true
    volumeMounts:
    - name: helm-deployment
      mountPath: "/data/secret"
      readOnly: false
  volumes:
  - name: helm-deployment
    secret:
      secretName: helm-deployment

"""
    } // kubernetes
  } // agent
  environment {
 
    def pom = readMavenPom file: "pom.xml"
    VERSION = pom.getVersion()
    }
  stages {

 

    stage('build:feature-dille') { 
      when { branch "feature-dille" }
       stages { 
          
           stage('build:package') {
             steps { container('jenkins-slave') {   
                    
                    sh "cd /data/secret"
                    sh "gcloud auth activate-service-account --key-file /data/secret/technology-platform-235213-978f6716dff6.json"	
					
                  sh """sed -i 's/tag:.*/tag: ${VERSION}/' dsp-menu-generation-service/values.yaml"""
                  sh "gcloud builds submit --config=cloudbuild.yaml --substitutions=_POM_XML_VERSION=${VERSION}"

				
                   sh "gcloud builds submit --config=cloudbuild.yaml --substitutions=_POM_XML_VERSION=${VERSION}"  				 
				   } }
            }
			stage('deploypodse') {
             steps { container('jenkins-slave') { 
			 
			  sh "gcloud container clusters get-credentials tretail-dev-com-kbe-marketplace-1 --region us-east4 --project tretail-dev-marketplace"
			  sh "cd dsp-menu-generation-service"
			  sh "helm  upgrade test-dsp-menu-generation-service dsp-menu-generation-service" } }
           }
			
		
			 	 
        } // stages
     } // build:development


   


  } // end stages
} // end pipeline
