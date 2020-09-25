// Pipeline to deploy nodejs application in docker containers.

def app_name = "nodejs"
def env = "${params.appENV}"
def BRANCH = "${params.appBRANCH}"
def git_url = "https://github.com/bindukan/nodejsproject.git"

pipeline {
	agent any
		parameters {
			string(
				name: 'appBRANCH',  
				defaultValue: "master",
				description: 'Write down your Tag/Branch to be deployed' )
			choice(
				name: 'appENV',  
				choices: "development\nproduction",
				description: 'Select the env to deploy' )
		}
		stages {
			stage('Checkout') {
				steps {
					checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '93559887-5cde-4a17-9279-e90ab6ac93f3', url: "${git_url}"]]])
				}
			}

			stage ('docker-Build') {
				steps {
				sh "docker build -t binduk17/${app_name}:v1.'$BUILD_NUMBER' ${WORKSPACE}/"

				}
			}
			stage('stop and remove exixting container') {
				steps{
					sh "docker rm -f ${app_name}"
				}
			}
			stage ('Run the Docker Container') {
				steps {
				sh "docker run -d -p 1337:8081/tcp -e NODE_ENV=${env} --name ${app_name} binduk17/${app_name}:v1.$BUILD_NUMBER"
				}
			}
		}
}  

