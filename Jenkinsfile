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
		   sh "env" 
		   sh "echo ${env.BUILD_ID}"
		   
		    def pom = readMavenPom file: 'pom.xml'
		// sh "echo ${pom.artifactId}"
		   sh "echo ${pom.version}"
            }

        }
          
   }
}
