pipeline {
    agent any
	
    tools {
        jdk 'Java17'
        maven 'Maven3'
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

	stage("Pom.xml INFO"){
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
	       
   }
}
