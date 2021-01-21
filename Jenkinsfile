pipeline {
    environment {
		PROJECT_ID = "sd-devops"
		APP_NAME = "sample-java-app"
		CLUSTER_NAME = "cluster-1"
		CLUSTER_ZONE = "us-central1-a"
		CREDENTIALS_ID = "sd-devops"
  	} 
	//agent any 2
  agent {
    kubernetes {
      label 'SpringBootRestApp'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  containers:
  - name: gradle
    image: gradle:3.5-jdk8-alpine
    command:
    - cat
    tty: true
"""
}
	}
  
    stages {
    stage('check_gradle_version') {
            steps {
		    container('gradle') {
                sh 'gradle -v'
                sh 'echo workspace is $WORKSPACE'
		    sh "ls -la ${pwd()}"
	          sh 'chmod 777 * '
		sh './gradlew compileJava'
            }
	  }
    }
 
	    
	  stage('Unit Test') {
			steps {
				container('gradle') {
					withMaven(maven: 'MAVEN-3.6.3') {
          	withSonarQubeEnv(installationName:'Sonarqube') {
							echo 'I am executing unit test'
			sh "ls -la ${pwd()}"
							sh 'mvn -f sample-java-app/pom.xml clean package'
			                                 
			
					               
				   	}
					}
				}
			}
		} 

		 stage('Code Quality') {
			steps {
				container('gradle') {
					withMaven(maven: 'MAVEN-3.6.3') {
						withSonarQubeEnv(installationName:'Sonarqube') {
							echo 'I am executing code quality using sonarqube'	
							sh 'mvn -f sample-java-app/pom.xml sonar:sonar'
						}
						sleep(60)
						timeout(time: 1, unit: 'MINUTES') {
							waitForQualityGate abortPipeline: true
						}			
					}
				}
			}   
		} 

		/*stage('Publish Package') {
            steps {
		    		container('gradle') {
				withMaven(maven: 'MAVEN-3.6.3') {
					echo 'I am executing build and push the artifact with unique name showing the branch from which it is generated, to Archiva'	
					sh 'mvn -X deploy:deploy-file -Dfile=sample-java-app/target/sample-0.0.1-SNAPSHOT.jar -DpomFile=sample-java-app/pom.xml -DrepositoryId=snapshots -Durl=http://35.188.92.10/repository/snapshots/'
				}		
            }
	    }
        }
		
		/* stage('Deploy Dev') {
			when { branch 'dev'}
            steps {
				echo "I am executing Deploy the artifact from Archiva to target dev environment. My artifact has a unique name which is automatically generated and deployed to target dev environment"
				echo "Work in progress"
            }
        }
		stage('Smoke Test'){
			when { branch 'dev'}
			steps {
				echo "I am executing Smoke Test on target dev environment post deployment"
				echo 'Work in progress'
			}
		} */
	}
}
