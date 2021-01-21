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
				-name: gradle
			image: gradle: 3.5 - jdk8 - alpine
			command:
				-cat
			tty: true
			"""
		}
	}
	stages {
		stage('test') {
			parallel {
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
								withSonarQubeEnv(installationName: 'Sonarqube') {
									echo 'I am executing unit test'
									sh "ls -la ${pwd()}"
									sh 'mvn -f sample-java-app/pom.xml clean package'
								}
							}
						}
					}
				}
			}
		}
		stage('Code Quality') {
			steps {
				container('gradle') {
					withMaven(maven: 'MAVEN-3.6.3') {
						withSonarQubeEnv(installationName: 'Sonarqube') {
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
	}
}
