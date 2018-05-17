LABEL = 'ubuntu'
buildJdk = 'JDK 1.8 (latest)'
buildJdk9 = 'JDK 1.9 (latest)'
buildJdk10 = 'JDK 10 (latest)'
buildMvn = 'Maven 3.5.2'
deploySettings = 'DefaultMavenSettingsProvider.1331204114925'


pipeline {
    agent {
        label "${LABEL}"
    }
    stages {
        stage('BuildAndDeploy') {
            steps {
                timeout(20) {
                    withMaven(maven: buildMvn, jdk: buildJdk,
                            mavenSettingsConfig: deploySettings,
                            mavenLocalRepo: ".repository",
                            options: [concordionPublisher(disabled: true), dependenciesFingerprintPublisher(disabled: true),
                                      findbugsPublisher(disabled: true), artifactsPublisher(disabled: true),
                                      invokerPublisher(disabled: true), jgivenPublisher(disabled: true),
                                      junitPublisher(disabled: true, ignoreAttachments: false),
                                      openTasksPublisher(disabled: true), pipelineGraphPublisher(disabled: true)]
                    )
                            {
                                // Run test phase / ignore test failures
                                sh "mvn -B -U -e -fae clean deploy"
                            }
                    // Report failures in the jenkins UI
                    //step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                }
            }
            post {
                success {
                    archiveArtifacts '**/target/*-site.xml,pom.xml'
                }
                failure {
                    notifyBuild("Failure in BuildAndDeploy stage")
                }
            }
        }

        stage('OtherJdks') {
            parallel {
                stage('JDK9') {
                    steps {
                        ws('JDK9') {
                            checkout scm
                            timeout(20) {
                                withMaven(maven: buildMvn, jdk: buildJdk9,
                                        mavenSettingsConfig: deploySettings,
                                        mavenLocalRepo: ".repository",
                                        options: [concordionPublisher(disabled: true), dependenciesFingerprintPublisher(disabled: true),
                                                  findbugsPublisher(disabled: true), artifactsPublisher(disabled: true),
                                                  invokerPublisher(disabled: true), jgivenPublisher(disabled: true),
                                                  junitPublisher(disabled: true, ignoreAttachments: false),
                                                  openTasksPublisher(disabled: true), pipelineGraphPublisher(disabled: true)]
                                )
                                        {
                                            // Run test phase / ignore test failures
                                            sh "mvn -B -U -e -fae clean verify"
                                        }
                            }
                        }
                    }
                }
                stage('JDK10') {
                    steps {
                        ws('JDK10') {
                            checkout scm
                            timeout(20) {
                                withMaven(maven: buildMvn, jdk: buildJdk9,
                                        mavenSettingsConfig: deploySettings,
                                        mavenLocalRepo: ".repository",
                                        options: [concordionPublisher(disabled: true), dependenciesFingerprintPublisher(disabled: true),
                                                  findbugsPublisher(disabled: true), artifactsPublisher(disabled: true),
                                                  invokerPublisher(disabled: true), jgivenPublisher(disabled: true),
                                                  junitPublisher(disabled: true, ignoreAttachments: false),
                                                  openTasksPublisher(disabled: true), pipelineGraphPublisher(disabled: true)]
                                )
                                        {
                                            // Run test phase / ignore test failures
                                            sh "mvn -B -U -e -fae clean verify"
                                        }

                            }
                        }
                    }
                }

            }
        }
    }
    post {
        unstable {
            notifyBuild("Unstable Build")
        }
        always {
            cleanWs deleteDirs: true, notFailBuild: true, patterns: [[pattern: '.repository', type: 'EXCLUDE'],[pattern: '.repository/**', type: 'EXCLUDE']]
        }
        success {
            script {
                def previousResult = currentBuild.previousBuild?.result
                if (previousResult && !currentBuild.resultIsWorseOrEqualTo(previousResult)) {
                    notifyBuild("Fixed")
                }
            }
        }
    }
}

// Send a notification about the build status
def notifyBuild(String buildStatus) {
    // default the value
    buildStatus = buildStatus ?: "UNKNOWN"

    def email = "notifications@archiva.apache.org"
    def summary = "${env.JOB_NAME}#${env.BUILD_NUMBER} - ${buildStatus} - ${currentBuild?.currentResult}"
    def detail = """<h4>Job: <a href='${env.JOB_URL}'>${env.JOB_NAME}</a> [#${env.BUILD_NUMBER}]</h4>
  <p><b>${buildStatus}</b></p>
  <table>
    <tr><td>Build</td><td><a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></td><tr>
    <tr><td>Console</td><td><a href='${env.BUILD_URL}console'>${env.BUILD_URL}console</a></td><tr>
    <tr><td>Test Report</td><td><a href='${env.BUILD_URL}testReport/'>${env.BUILD_URL}testReport/</a></td><tr>
  </table>
  """

    emailext(
            to: email,
            subject: summary,
            body: detail,
            mimeType: 'text/html'
    )
}

// vim: et:ts=4:sw=4:ft=groovy
