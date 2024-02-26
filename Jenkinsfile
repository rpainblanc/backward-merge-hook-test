import groovy.json.JsonSlurperClassic
import org.jenkinsci.plugins.pipeline.modeldefinition.Utils

pipeline {
    agent {
        docker {
            image "${DKU_DOCKER_JENKINS_BUILDER_IMAGE}"
            label 'docker-builder'
        }
    }
    parameters {
        booleanParam(name: 'RUN_SONAR_BY_MANUAL_TRIGGER', defaultValue: false, description: 'Whether to run Sonar scan')
    }
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '60'))
        disableConcurrentBuilds()
        // 4 hours should be more than enough, almost all builds execute within 2 hours
        timeout(time: 4, unit: 'HOURS')
    }
    stages {
        stage('Check mergeability status') {
            steps {
                script {
                    if (env.BRANCH_NAME != 'master') { // only check if different than master, everything can go there
                        // The logic is the same for the pre-merge-commit hook, just check if a wrong commit is in the source branch
                        try {
                            sh "chmod u+x ./pre-merge-commit"
                            if (env.CHANGE_TARGET) {
                                sh "./pre-merge-commit ${env.BRANCH_NAME} ${env.CHANGE_TARGET}"
                            } else {
                                // On purpose, current == target, just to check in current history
                                sh "./pre-merge-commit ${env.BRANCH_NAME} ${env.BRANCH_NAME}"
                            }
                        } catch (e) {
                            notifyBuild("POTENTIAL_MERGE_ISSUE")
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                notifyBuild(currentBuild.result)
            }
        }
    }
}

// Return the name of the branch the job is running on. Depending on how the job is configured
// in Jenkins, this information come from the following environment variables:
//  - CHANGE_BRANCH: if the job is configured with GitHub organization and it's a PR
//  - BRANCH_NAME || GIT_BRANCH:
//    - if the job is configured with GitHub organization and it's a simple branch
//    - or if the branch is set in the SCM configuration of the job
def getBranchName() {
    // Check use-cases in this order: CHANGE_BRANCH then (BRANCH_NAME || GIT_BRANCH)
    String branchName = env.CHANGE_BRANCH
    if (branchName == null) {
        branchName = env.BRANCH_NAME
    }
    if (branchName == null) {
        branchName = env.GIT_BRANCH
    }

    return branchName.replaceAll(/^origin\//, '')
}

def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESS'

    // Default values
    def colorCode = '#FF0000'

    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    // Override default values based on build status
    if (buildStatus == 'STARTED') {
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        colorCode = '#00FF00'
    } else {
        colorCode = '#FF0000'
    }

    // Send notifications
    String branchName = getBranchName()
    if (branchName != null && (branchName.startsWith("release") || branchName.startsWith("single-release") || branchName == "master")) {
        // def slackChannel = (buildStatus == 'SUCCESS') ? '#rd-notifications' : '#rd-general'
        def slackChannel = '#rd-jenkins-tests'
        slackSend color: colorCode, message: summary, notifyCommitters: true, channel: slackChannel
    }
}

