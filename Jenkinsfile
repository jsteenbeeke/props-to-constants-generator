@Library(value = 'jenkins-shared-library@main', changelog = false)
import com.jeroensteenbeeke.hyperion.*

Hyperion hyperion = new Hyperion(this)
def pollInterval = hyperion.scmPollInterval()

pipeline {
    agent {
        docker {
            image 'registry.jeroensteenbeeke.nl/maven:latest'
            label 'docker'
            alwaysPull true
        }
    }

    triggers {
        pollSCM(pollInterval)
        upstream(upstreamProjects: 'jeroensteenbeeke-parent', threshold: hudson.model.Result.SUCCESS)
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        disableConcurrentBuilds()
    }

    environment {
        MAVEN_DEPLOY_USER = credentials('MAVEN_DEPLOY_USER')
        MAVEN_DEPLOY_PASSWORD = credentials('MAVEN_DEPLOY_PASSWORD')
    }


    stages {

        stage('Maven') {
            steps {
                sh 'mvn -ntp -B -U clean verify package'
            }
        }

        stage('Deploy') {
            steps {
                mavenDeploy deployUser: env.MAVEN_DEPLOY_USER,
                        deployPassword: env.MAVEN_DEPLOY_PASSWORD
            }
        }


    }

    post {
        always {
            script {
                if (currentBuild.result == null) {
                    currentBuild.result = 'SUCCESS'
                }
            }

            step([$class                  : 'Mailer',
                  notifyEveryUnstableBuild: true,
                  sendToIndividuals       : true,
                  recipients              : 'j.steenbeeke@gmail.com'
            ])
        }
    }

}
