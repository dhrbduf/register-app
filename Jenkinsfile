pipeline {
    agent any
	
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
	    APP_NAME = "register-app-pipeline"
  	    RELEASE = "1.0.0"
	    DOCKER_USER = "dhrbduf"
	    DOCKER_PASS = 'dockerhub'
	    IMAGE_NAME = "${DOCKER_USER}"+"/"+"${APP_NAME}"
	    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
	
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/dhrbduf/register-app'
                }
        }

        stage("Build Application"){
                steps {
                    sh "mvn clean package"
                }
       }

        stage("Test Application"){
               steps {
                     sh "mvn test"
               }
       }

        stage("SonarQube Analysis"){
               steps {
	               script {
		                withSonarQubeEnv(credentialsId: 'jenkins-sonar-token') { 
                        sh "mvn sonar:sonar"
		                }
	               }	
               }
       }

	stage("Quality Gate"){
       	    steps {
            	   script {
               	     	waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar-token'
                   }
            }

        }

	stage("Pom.xml Info"){
       	    steps {
     
		   script { 
			   sh "env" 
			   sh "echo ${env.BUILD_ID}"
			   sh "echo $currentBuild.number"
				   
		        // def VERSION = readMavenPom().getVersion()
			   def VERSION = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true)
			   sh "echo $VERSION"
	
			   def ARTIFACTID = sh(script: 'mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout', returnStdout: true)   
			   sh "echo $ARTIFACTID"   

			   def GROUPID = sh(script: 'mvn help:evaluate -Dexpression=project.groupId -q -DforceStdout', returnStdout: true)
			   sh "echo $GROUPID"  
			   }
            }

        }

	stage("Build & Push Docker Iamge"){
       	    steps {
            	   script {
               	     	docker.withRegistry('',DOCKER_PASS) {
				docker_image = docker.build "${IMAGE_NAME}"
                   	}

			docker.withRegistry('',DOCKER_PASS) {
				docker_image.push("${IMAGE_TAG}")
				docker_image.push('latest')
                   	}	   
            	   }

            }       
       }

       stage("Trivy Scan") {
           steps {
               script {
            	    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image dhrbduf/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }

       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://10.50.6.98:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
	    
	    
    }
}
