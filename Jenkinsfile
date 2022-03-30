def rtServer, rtDocker, buildInfo, privateDockerRegistry

pipeline {
  agent any
  parameters {
        string (name: 'DOCKER_REPO', defaultValue: 'docker-local', description: 'Docker repository for pull/push')
    }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    CI = true
    ARTIFACTORY_ACCESS_TOKEN = credentials('jfrog-access-token')
	ARTIFACTORY_LOGIN = credentials('jfrog-creds')
  }
  stages {
    stage('Build') {
      steps {
        sh './mvnw clean install'
      }
    }
	stage('Setup'){
            steps {
                script{
                    rtServer = Artifactory.newServer url: 'https://jfrogfreerepo.jfrog.io/artifactory/', username: "${ARTIFACTORY_LOGIN_USR}", password: "${ARTIFACTORY_LOGIN_PSW}"
                    rtDocker = Artifactory.docker server: rtServer
                    privateDockerRegistry = 'jfrogfreerepo.jfrog.io'
                }
            }
        }
	stage('Create Image'){
            steps {
                script{
                    def dockerImage = docker.build("${privateDockerRegistry}/${params.DOCKER_REPO}/build-${JOB_NAME}:${BUILD_NUMBER}")
                }
            }
        }
    stage('Deploy Docker Image to Artifactory'){
            steps {
                script {
                    buildInfo = rtDocker.push "${privateDockerRegistry}/${params.DOCKER_REPO}/build-${JOB_NAME}:${BUILD_NUMBER}", "${params.DOCKER_REPO}"
                }
            }
        }
	stage('Publish'){
            steps {
                script {
                    rtServer.publishBuildInfo buildInfo
                }
            }
        }
    }
}
