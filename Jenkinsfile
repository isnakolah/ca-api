// *********************** Jenkins file for staging server********************

pipeline {
	agent any
	environment{
		registryfront = "poneahealth/epharmacy"
		registryapi = "poneahealth/epharmacy"
		registryCredential = 'dockerhubponea'
		REGISTRY_ADDRESS = "https://registry.hub.docker.com"
		dockerImage1 = ''
		dockerImage2 = ''
		COMPOSE_FILE = "docker-compose.yml"
		REGISTRY_AUTH = credentials("dockerhubponea")
	}
	
	stages {
		stage('Verify') {
			steps {
				sh '''
					docker version
					docker-compose version
				'''
			}
		}

		stage('clean') {
		   steps {
			 sh 'sh dockerclean.sh'
		   }
		}


		stage('build') {
			steps {
				sh 'docker-compose build'
			}
		}

		stage("Determine new version") {

			  steps {
				  script {
					  env.DEPLOY_VERSION = sh(returnStdout: true, script: "docker run --rm -v '${env.WORKSPACE}':/repo:ro softonic/ci-version:0.1.0").trim()
					  env.DEPLOY_MAJOR_VERSION = sh(returnStdout: true, script: "echo '${env.DEPLOY_VERSION}' | awk -F'[ .]' '{print \$1}'").trim()
					  env.DEPLOY_COMMIT_HASH = sh(returnStdout: true, script: "git rev-parse HEAD | cut -c1-7").trim()
					  env.DEPLOY_BUILD_DATE = sh(returnStdout: true, script: "date -u +'%Y-%m-%dT%H:%M:%SZ'").trim()
					  env.IS_NEW_VERSION = sh(returnStdout: true, script: "[ '${env.DEPLOY_VERSION}' ] && echo 'YES'").trim()
				  }
			  }
		}

		 /* stage("Create new version") {
		 	when {
		 		environment name: "IS_NEW_VERSION", value: "YES"
		 	}
		 	steps {
		 		script {
				  
		 				sh """
		 					git config user.email "ci-sandeep@email.com"
		 					git config user.name "Jenkins"
		 					git tag -a "v${env.DEPLOY_VERSION}" \
		 						-m "Generated by: ${env.JENKINS_URL}" \
		 						-m "Job: ${env.JOB_NAME}" \
		 						-m "Build: ${env.BUILD_NUMBER}" \
		 						-m "Env Branch: ${env.BRANCH_NAME}"
		 					git push origin "v${env.DEPLOY_VERSION}"
		 				"""
		 				   }
		 		}
		 	} */

		stage('Push Image') {
		  steps{    script {
			  docker.withRegistry( '', registryCredential ) {
			    sh "docker-compose -f ${env.COMPOSE_FILE} build"
				//sh "docker-compose -f ${env.COMPOSE_FILE} push"
				//dockerImage1.push()
				//dockerImage2.push()
			  }
			}
		  }
		}
				stage('Deploy') {
			steps {
				sh 'docker-compose kill'
				sh 'docker-compose up -d'
			}
		}

	}
}
