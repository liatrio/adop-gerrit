//ldop-gerrit/Jenkinsfile
pipeline {
    agent none

    stages {
      stage('hadolint-lint'){
          agent {
              docker {
                  image "lukasmartinelli/hadolint"
                  args "-u root"
              }
          }
          steps {
              sh 'hadolint Dockerfile || true'
          }
         post {
              failure {
                  script {
                      subject = "failure: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
                  }
                  slackSend (color: '#FF0000', message: "${subject} (${env.BUILD_URL})")
              }
          }
      }
      stage('dockerlint-lint'){
          agent {
              docker {
                  image "redcoolbeans/dockerlint"
              }
          }
          steps {
              sh 'dockerlint -f Dockerfile || true'
          }
         post {
              failure {
                  script {
                      subject = "failure: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
                  }
                  slackSend (color: '#FF0000', message: "${subject} (${env.BUILD_URL})")
              }
          }
      }
      stage('dockerfile-lint'){
          agent {
              docker {
                  image "projectatomic/dockerfile-lint"
                  args "-u root"
              }
          }
          steps {
              sh 'dockerfile_lint -f Dockerfile || true'
          }
         post {
              failure {
                  script {
                      subject = "failure: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
                  }
                  slackSend (color: '#FF0000', message: "${subject} (${env.BUILD_URL})")
              }
          }
      }
      stage('ldop-gerrit-validate'){
          agent any
          steps {
              sh "echo \$(git tag --sort version:refname | tail -1) > result"
              script {
                  TAG = readFile 'result'
                  TAG = TAG.trim()
              }
              doesVersionExist('liatrio', 'ldop-gerrit', "${TAG}") 
          }
      }
      /*stage('ldop-gerrit-build'){
          agent any
          steps {
              sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD}"
              sh "docker build -t chadliatrio/ldop-gerrit:${env.BRANCH_NAME} ."
              sh "docker push chadliatrio/ldop-gerrit:${env.BRANCH_NAME}"
          }
      }*/
      stage('ldop-integration-testing'){
          agent any
          steps {
              git branch: 'master', url: 'https://github.com/liatrio/ldop-docker-compose'
              sh "whoami"
              sh "ls -al"
              sh "pwd"
			  sh 'touch /var/jenkins_home/workspace/gerrit_LDOP-158-jenkinsfile-PHWFZQFZEJ4N5CBJGHI44KBLM2THAGGOAXOR5PB3CBJ3HVGSPFFA/destination.txt'
			  sh 'chmod 777 /var/jenkins_home/workspace/gerrit_LDOP-158-jenkinsfile-PHWFZQFZEJ4N5CBJGHI44KBLM2THAGGOAXOR5PB3CBJ3HVGSPFFA/destination.txt'
              testSuite("ldop-gerrit", "${TAG}")
          }
      }
      stage('ldop-image-deploy'){
          agent any
          steps {
              sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD}"
              sh "docker tag chadliatrio/ldop-gerrit:${env.BRANCH_NAME} chadliatrio/ldop-gerrit:${TAG}"
              sh "docker push chadliatrio/ldop-gerrit:${TAG}"
          }
      }
  }
}
