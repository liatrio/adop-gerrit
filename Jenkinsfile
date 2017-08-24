//ldop-gerrit/Jenkinsfile

pipeline {
    agent none

    stages {
        stage('pipeline-init') {
            agent any 

            steps {
                script {
                    STAGE = ""
                    STATUS = ""
                    CHANGED = "NO"

                    CHANGES = ""
                    SUBJECT = ""

                    COLOR = ""
                }
            }
        }
        stage('ldop-gerrit-validate') {
            agent any
            steps {
                sh "git fetch"
                sh "echo \$(git tag -l | sort -V | tail -1) > result"
                script {
                    TAG = readFile 'result'
                    TAG = TAG.trim()
                  
                    if (!(TAG ==~ /^[0-9]+\.[0-9]+\.[0-9]+$/)) {
                        error("Invalid Git tag format! Aborting...")
                    }

                    if (doesVersionExist('liatrio', 'ldop-gerrit', "${TAG}")) {
                        error("LDOP Gerrit version already exists! Aborting...")
                    }
                }
            }
            post {
                success {
                    script {
                        STATUS = "SUCCESS"
                    }
                }
                failure {
                    script {
                        STATUS = "FAILURE"
                    }
                }
                changed {
                    script {
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'ldop-gerrit-validate'" } }
            }
        }
        stage('hadolint-lint') {
            agent {
                docker {
                    image "lukasmartinelli/hadolint"
                    args "-u root"
                }
            }
            steps {
                script { CHANGED = "NO" }
                sh 'hadolint Dockerfile || true'
            }       
            post {
                success {
                    script {
                        STATUS = "SUCCESS"
                    }
                }
                failure {
                    script {
                        STATUS = "FAILURE"
                    }
                }
                changed {
                    script {
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'hadolint-lint'" } }
            }
        }
        stage('dockerlint-lint') {
            agent {
                docker {
                    image "redcoolbeans/dockerlint"
                }
            }
            steps {
                script { CHANGED = "NO" }
                sh 'dockerlint -f Dockerfile || true'
            }
            post {
                success {
                    script {
                        STATUS = "SUCCESS"
                    }
                }
                failure {
                    script {
                        STATUS = "FAILURE"
                    }
                }
                changed {
                    script {
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'dockerlint-lint'" } }
            }
        }
        stage('dockerfile-lint') {
            agent {
                docker {
                    image "projectatomic/dockerfile-lint"
                    args "-u root"
                }
            }
            steps {
                script { CHANGED = "NO" }
                sh 'dockerfile_lint -f Dockerfile || true'
            }
            post {
                success {
                    script {
                        STATUS = "SUCCESS"
                    }
                }
                failure {
                    script {
                        STATUS = "FAILURE"
                    }
                }
                changed {
                    script {
                        CHANGED = "YES"
                    }
                }
                always { script { SUBJECT = "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} at 'dockerfile-lint'" } }
            }
        }
    }
    post {
        always {
            script {
                if (CHANGED == "YES") {
                    if (STATUS == "SUCCESS") {
                        COLOR = "00FF00"
                    } else {
                        COLOR = "FF0000"
                    }

                    RESULT = formatSlackOutput(SUBJECT, env.JOB_URL, currentBuild.changeSets, STATUS)

                    slackSend (color: "#${COLOR}", message: "${RESULT}")
                }
            }
        }
    }
}
