pipeline {
	agent any
	parameters {
		string(defaultValue: "USERNAME", description: 'enter username: ', name: 'username')
		string(defaultValue: "PASSWORD", description: 'enter password: ', name: 'password')
		string(defaultValue: "ldop-gerrit", description: 'what repo?', name: 'repository')
		string(defaultValue: "IMAGE", description: 'what image?', name: 'image')
		string(defaultValue: "master", description: 'what branch is this?', name: 'branch')
	}
	stages {
		stage('validate'){
			doesVersionExist(${params.username}, ${params.password}, ${params.repository}, ${params.image}, '')
		}
		stage('build'){
			steps {
				sh 'docker build -t liatrio/${params.image}:${params.branch} .'
				sh 'docker push liatrio/${params.image}:${params.branch}'
				sh './test/validation/validation.sh'
				testSuite(${params.image}, ${params.branch})
			}
		}
		stage('test'){
			testSuite(${params.image}, ${params.branch})
		}
		stage('deploy'){
			steps {
				sh 'docker tag liatrio/${IMAGE_NAME}:${TOPIC} liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
				sh 'docker push liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
			}
		}
	}
}
