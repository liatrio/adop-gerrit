pipeline {
	agent none
	stages {
		stage('Dockerfile-lint'){
			agent {
				docker {
					image "lukasmartinelli/hadolint"
				}
			}
			steps {
				sh ''
			}
		}
		stage('${params.image}-validate'){
			agent any
			steps {
				git branch: '${params.branch}', url: ''
				doesVersionExist(${params.username}, ${params.password}, ${params.repository}, ${params.image}, '')
				getLatestVersion(${params.username}, ${params.password}, ${params.repository}, ${params.image})
			}
		}
		stage('${params.image}-build'){
			agent any
			steps {
				sh 'docker build -t liatrio/${params.image}:${params.branch} .'
				sh 'docker push liatrio/${params.image}:${params.branch}'
			}
		}
		stage('ldop-integration-testing'){
			agent any
			steps {
				testSuite(${params.image}, ${params.branch})
			}
		}
		stage('ldop-image-deploy'){
			agent any
			steps {
				sh 'docker tag liatrio/${IMAGE_NAME}:${TOPIC} liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
				sh 'docker push liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
			}
		}
	}
}
