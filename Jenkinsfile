/*
* This program and the accompanying materials are made available under the terms of the
* Eclipse Public License v2.0 which accompanies this distribution, and is available at
* https://www.eclipse.org/legal/epl-v20.html
*
* SPDX-License-Identifier: EPL-2.0
*
* Copyright Contributors to the Zowe Project.
*
*/

/**
 * List of people who will get all emails for master builds
 */
def MASTER_RECIPIENTS_LIST = "cc:Pavel.Zlatnik@broadcom.com"

/**
 * The name of the master branch
 */
//def MASTER_BRANCH = "master"
def MASTER_BRANCH = "initDevPipeline1"

/**
* Is this a release branch? Temporary workaround that won't break everything horribly if we merge.
*/
def RELEASE_BRANCH = false
/**
 * The result string for a successful build
 */
def BUILD_SUCCESS = 'SUCCESS'

/**
 * The result string for an unstable build
 */
def BUILD_UNSTABLE = 'UNSTABLE'

/**
 * The result string for a failed build
 */
def BUILD_FAILURE = 'FAILURE'

/**
 * The user's name for git commits
 */
def GIT_USER_NAME = 'zowe-robot'

/**
 * The user's email address for git commits
 */
def GIT_USER_EMAIL = 'zowe.robot@gmail.com'

/**
 * The base repository url for github
 */
def GIT_REPO_URL = 'https://github.com/zowe/vtl-cli.git'

/**
 * The credentials id field for the authorization token for GitHub stored in Jenkins
 */
def GIT_CREDENTIALS_ID = 'zowe-robot-github'

/**
 * A command to be run that gets the current revision pulled down
 */
def GIT_REVISION_LOOKUP = 'git log -n 1 --pretty=format:%h'

/**
 * The credentials id field for the artifactory username and password
 */
def ARTIFACTORY_CREDENTIALS_ID = 'GizaArtifactory'

/**
 * The email address for the artifactory
 */
def ARTIFACTORY_EMAIL = GIT_USER_EMAIL

/**
* The VTL CLI Bundle Version to deploy to Artifactory
*/
def VTL_CLI_BUNDLE_VERSION = "0.1.0-SNAPSHOT"

/**
*  The Artifactory Server to deploy to.
*/ 
def ARTIFACTORY_SERVER = "gizaArtifactory"

/**
* The target repository for VTL CLI Package SNAPSHOTs
*/ 
def ARTIFACTORY_SNAPSHOT_REPO = "libs-snapshot-local"

/**
* Target Repository for VTL CLI Package Releases
*/ 
def ARTIFACTORY_RELEASE_REPO = "libs-release-local"


// Setup conditional build options. Would have done this in the options of the declarative pipeline, but it is pretty
// much impossible to have conditional options based on the branch :/
def opts = []

if (BRANCH_NAME == MASTER_BRANCH) {
    // Only keep 20 builds
    opts.push(buildDiscarder(logRotator(numToKeepStr: '20')))

    // Concurrent builds need to be disabled on the master branch because
    // it needs to actively commit and do a build. There's no point in publishing
    // twice in quick succession
    opts.push(disableConcurrentBuilds())
} else {
    if (BRANCH_NAME.equals("1.0.0")){
        RELEASE_BRANCH = true
    }
    // Only keep 5 builds on other branches
    opts.push(buildDiscarder(logRotator(numToKeepStr: '5')))
}
properties(opts)

pipeline {
    agent {
        label 'apiml-jenkins-agent'
    }

    options {
        timestamps ()
    }

    stages {
        // Stage 1
        stage ('Build') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh 'chmod +x ./gradlew'
                    sh './gradlew build'
                }
            }
        }
        // Stage 2
        stage ('Create executable scripts') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh 'chmod +x scripts/create_executable_jars.sh'
                    sh 'scripts/create_executable_jars.sh'                    
                }
            }
        }
        // Stage 3
        stage ('Zip/tar files') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh 'cd build; tar -czvf vtl.tar.gz vtl-cli.jar vtl zos/vtl; cd ..'                    
                }
            }
        }
        // Stage 4
        stage ('Archive artifact') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    archiveArtifacts artifacts: 'build/vtl.tar.gz'                   
                }
            }
        }
        // Stage 5
        stage('Publish snapshot version to Artifactory for master') {
                    when {
                        expression {
                            return BRANCH_NAME.equals(MASTER_BRANCH);
                        }
                    }
                    steps {
                        timeout(time: 5, unit: 'MINUTES' ) {
                            script {
                            def server = Artifactory.server ARTIFACTORY_SERVER
                            def targetVersion = VTL_CLI_BUNDLE_VERSION
                            def targetRepository = targetVersion.contains("-SNAPSHOT") ? ARTIFACTORY_SNAPSHOT_REPO : ARTIFACTORY_RELEASE_REPO
                            def uploadSpec = """{
                            "files": [{
                                "pattern": "build/vtl.tar.gz",
                                "target": "${targetRepository}/org/zowe/vtl-cli/zowe-cli-package/${targetVersion}/"
                            }]
                            }"""
                            def buildInfo = Artifactory.newBuildInfo()
                            server.upload spec: uploadSpec, buildInfo: buildInfo
                            server.publishBuildInfo buildInfo
                            }
                        }
                    }
        }
        // Stage 6
        stage ('Codecov') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Codecov', usernameVariable: 'CODECOV_USERNAME', passwordVariable: 'CODECOV_TOKEN')]) {
                    sh 'curl -s https://codecov.io/bash | bash -s'
                }
            }
        }
    }

    post {
        
        //success {
        //    archiveArtifacts artifacts: 'build/vtl.tar.gz'
        //}
        always{
            //script {
            //    def buildStatus = currentBuild.currentResult
            //    echo "Build status for current build is ${buildStatus} ."
            //}
            script {
                def buildStatus = currentBuild.currentResult
                try {
                    def recipients = "${MASTER_RECIPIENTS_LIST}"

                    def subject = "${currentBuild.currentResult}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
                    def consoleOutput = """
                    <p>Branch: <b>${BRANCH_NAME}</b></p>
                    <p>Check console output at "<a href="${RUN_DISPLAY_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>
                    """

                    if (details != "") {
                        echo "Sending out email with details"
                        emailext(
                                subject: subject,
                                to: recipients,
                                body: "${consoleOutput}",
                                recipientProviders: [[$class: 'DevelopersRecipientProvider'],
                                                        [$class: 'UpstreamComitterRecipientProvider'],
                                                        [$class: 'CulpritsRecipientProvider'],
                                                        [$class: 'RequesterRecipientProvider']]
                        )
                    }
                } catch (e) {
                    echo "Experienced an error sending an email for a ${buildStatus} build"
                    currentBuild.result = buildStatus
                }
            }
        }
    }
}