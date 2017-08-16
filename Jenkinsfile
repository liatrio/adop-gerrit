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
		stage('${params.image}-0-validate'){
			doesVersionExist(${params.username}, ${params.password}, ${params.repository}, ${params.image}, '')
			getLatestVersion(${params.username}, ${params.password}, ${params.repository}, ${params.image})
			slackSend baseUrl: "https://liatrio.slack.com", channel: "#ldop", message: "Validation for ${params.image} failed"
		}
		stage('${params.image}-1-build'){
			steps {
				sh 'docker build -t liatrio/${params.image}:${params.branch} .'
				sh 'docker push liatrio/${params.image}:${params.branch}'
			}
			slackSend baseUrl: "https://liatrio.slack.com", channel: "#ldop", message: "Build for ${params.image} failed"
		}
		stage('ldop-integration-testing'){
			testSuite(${params.image}, ${params.branch})
			slackSend baseUrl: "https://liatrio.slack.com", channel: "#ldop", message: "Integration for ${params.image} failed"
		}
		stage('ldop-image-deploy'){
			steps {
				sh 'docker tag liatrio/${IMAGE_NAME}:${TOPIC} liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
				sh 'docker push liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
			}
			slackSend baseUrl: "https://liatrio.slack.com", channel: "#ldop", message: "Deployment for ${params.image} failed"
		}
	}
}
