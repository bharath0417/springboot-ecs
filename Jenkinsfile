pipeline {
    agent any
	environment {
		registry = 'ecs-task:v_$BUILD_NUMBER'
		registryUrl='421168438563.dkr.ecr.ap-south-1.amazonaws.com'
		registryCredentials = 'ecr:ap-south-1:docker_credentials'
		dockerImage = ''
	}
    stages {
        stage('CleanWorkspace') {
            steps {
                cleanWs()
            }
        }
        stage ('SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                	branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, 
                	extensions: [], 
                	submoduleCfg: [], 
                	userRemoteConfigs: [[url: 'https://github.com/bharath0417/springboot-hello.git']]])
            }
        }
        stage ('Build Artifact') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage ('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        stage ('Docker Push') {
            steps {
                script {
                    docker.withRegistry(registryUrl,registryCredentials) {
                        dockerImage.push()
                    }
                }
            }
        }
                stage ('ecs-deploy') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'docker_credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        docker.withRegistry(registryUrl,registryCredentials) {
                            sh """
                                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                                export AWS_DEFAULT_REGION=ap-south-1
                                chmod +x deploy.sh
                                ./deploy.sh"""
                        }
                    }
                }
            }
        }
    }
}
