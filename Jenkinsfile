//ldop-gerrit/Jenkinsfile
pipeline {
    agent none

    stages {
      stage('get-topic'){
        agent any
        steps {
            sh "env"
            sh "printenv"
            sh "git branch"
            sh "echo \$(git log -n 1 --pretty=%d HEAD | sed -n 's/.*origin\\/\\([A-Za-z0-9-]*\\))/\\1/p') > result"
            script {
                TOPIC = readFile 'result'
                TOPIC = TOPIC.trim()
            }
            echo "${TOPIC}"
        }
      }
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
              println doesVersionExist('liatrio', 'ldop-gerrit', "${TAG}") 
              getLatestVersion('liatrio', 'ldop-gerrit')
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
      stage('ldop-gerrit-build'){
          agent any
          steps {
              //need sh " TOPIC="${GIT_BRANCH#*/}" "
              //need sh "docker build -t liatrio/ldop-gerrit:${topic?} ."
              //need sh 'docker push liatrio/ldop-gerrit:${topic?}'
              echo 'build gerrit'
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
      stage('ldop-integration-testing'){
          agent any
          steps {
              git branch: 'master', url: 'https://github.com/liatrio/ldop-docker-compose'
              //need TOPIC="${TOPIC#*/}"
              //testSuite("ldop-gerrit", "${tag}")
              echo 'run test suite'
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
      stage('ldop-image-deploy'){
          agent any
          steps {
              //need TOPIC="${TOPIC#*/}"
              //sh 'docker tag liatrio/ldop-gerrit:${tag} liatrio/ldop-gerrit:${IMAGE_VERSION}'
              //sh 'docker push liatrio/${IMAGE_NAME}:${IMAGE_VERSION}'
              echo 'deploying stage'
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
  }
}
