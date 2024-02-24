library 'jenkins-shared-library@main'

pipeline {
    agent {
        docker {
            image 'registry.jeroensteenbeeke.nl/maven:latest'
            label 'docker'
            alwaysPull true
        }
    }

    triggers {
        pollSCM('H/5 * * * *')
        upstream(upstreamProjects: 'andalite/master,lux,vavr-hamcrest,jakartafied-projects,fansasstic-maven-plugin', threshold: hudson.model.Result.SUCCESS)
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
                addGitMetadata folder: 'core', package: 'com.jeroensteenbeeke.hyperion'
                sh 'mvn -B -U clean verify install -P-disable-slow-tests'
            }
        }

        stage('Deploy') {
            when {
                expression {
                    (currentBuild.result == 'SUCCESS' || currentBuild.result == null) && (env.BRANCH_NAME == 'master')
                }
            }

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
