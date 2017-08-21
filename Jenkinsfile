//ldop-gerrit/Jenkinsfile
pipeline {
    agent none

    stages {
        stage('pipeline-init'){
            agent any 

            steps {
                script {
                    STAGE = ""
                    STATUS = ""
                    CHANGED = "NO"

                    SUBJECT = ""

                    COLOR = "FF0000"
                }
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
                sh 'hadolint Dockerfile'
            }       
            post {
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'hadolint-lint'" } }
            }
        }
        stage('dockerlint-lint'){
            agent {
                docker {
                    image "redcoolbeans/dockerlint"
                }
            }
            steps {
                sh 'env'
                sh 'dockerlint -f Dockerfile || true'
            }
            post {
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'dockerlint-lint'" } }
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
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'dockerfile-lint'" } }
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
            post {
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'ldop-gerrit-validate'" } }
            }
        }
        stage('ldop-gerrit-build'){
            agent any
            steps {
                // sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD}"
                sh "docker build -t liatrio/ldop-gerrit:${env.BRANCH_NAME} ."
                sh "docker push liatrio/ldop-gerrit:${env.BRANCH_NAME}"
            }
            post {
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'ldop-gerrit-build'" } }
            }
        }
        stage('ldop-integration-testing'){
            agent {
              docker {
                image "hashicorp/terraform:full"
                args "-u root"
              }
            }
            steps {
                git branch: 'master', url: 'https://github.com/liatrio/ldop-docker-compose'
                sh "echo \$(pwd) > result"
                script {
                  DIR = readFile 'result'
                  DIR = DIR.trim()
                }
                testSuite("ldop-gerrit", "${TAG}", "${DIR}")
                sh "export TF_VAR_branch_name=\"${env.BRANCH_NAME}\""
                sh '''sed -i 's/timeout 30m/timeout -t 1800/g' test/integration/run-integration-test.sh &&
                      bash test/validation/validation.sh &&
                      cd test/integration/ &&
                      terraform init &&
                      bash run-integration-test.sh'''
            }
            post {
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'ldop-integration-testing'" } }
            }
        }
        stage('ldop-image-deploy'){
            agent any
            steps {
                // sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD}"
                sh "docker tag liatrio/ldop-gerrit:${env.BRANCH_NAME} liatrio/ldop-gerrit:${TAG}"
                sh "docker push liatrio/ldop-gerrit:${TAG}"
            }
            post {
                success {
                    script {
                        COLOR = "00FF00"
                        STATUS = "SUCCESS"
                    }
                }
                failure { script { STATUS = "FAILURE" } }
                changed {
                    script {
                        COLOR = "FFA500"
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'ldop-image-deploy'" } }
            }
        }
    }
    post {
        always {
            script {
                slackSend (color: "#${COLOR}", message: "${SUBJECT}. Result: ${STATUS}. (${env.JOB_URL})")
            }
        }
    }
}
