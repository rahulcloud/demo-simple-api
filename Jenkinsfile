def rtServer, rtDocker, buildInfo, privateDockerRegistry

pipeline {
  agent any
  parameters {
        string (name: 'DOCKER_REPO', defaultValue: 'docker-local', description: 'Docker repository for pull/push')
	booleanParam(name: "SKIPK8S", defaultValue: false, description: 'Check the box to skip k8s stage')
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
    /**stage('Deploying App to Kubernetes') {
            steps {
               script {
                   kubernetesDeploy(configs: "k8s-deployment.yaml", kubeconfigId: "kubernetes")
                }
            }
        }*/
	stage('Update K8s docker image with latest build'){
	    when {
            expression {
                !params.SKIPK8S
            }
        }
            steps {
                script {
				    def IMAGENAME = "${privateDockerRegistry}/${params.DOCKER_REPO}/build-${JOB_NAME}:${BUILD_NUMBER}"
                    sh "sed -i 's|IMAGENAME|${IMAGENAME}|g' k8s-deployment.yaml"
                }
            }
        }
	/**stage('Docker Deploy Dev'){
	    when {
            expression {
                !params.SKIPK8S
            }
        }
        steps{
            sshagent(['minikube-server']) {
                sh "scp -o StrictHostKeyChecking=no k8s-deployment.yaml ubuntu@3.88.222.101:/home/ubuntu/"
				    script{
			            try{
				            sh "ssh ubuntu@3.88.222.101 kubectl apply -f ."
				        }catch(error){
				    	    sh "ssh ubuntu@3.88.222.101 kubectl create -f ." 
				        }
				    }
					
                }
            }
        }*/
	stage('Deploying App to Kubernetes') {
	    when {
            expression {
                !params.SKIPK8S
            }
        }
            steps {
               script {
                   kubernetesDeploy(configs: "k8s-deployment.yaml", kubeconfigId: "kubernetes")
                }
            }
        }
    }
}
