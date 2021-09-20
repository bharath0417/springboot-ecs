pipeline {
    agent any
tools { 
        maven 'Maven'
    }
environment {
		registry = 'spring-ecs:v_$BUILD_NUMBER'
		registryUrl='421168438563.dkr.ecr.ap-south-1.amazonaws.com'

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
                	userRemoteConfigs: [[url: 'https://github.com/bharath0417/springboot-ecs.git']]])
            }
        }
        stage ('Build Artifact') {
            steps {
                sh 'mvn clean install'
            }
        }
      stage("Build the docker image"){
            steps{
                sh 'docker image build -t 421168438563.dkr.ecr.ap-south-1.amazonaws.com/spring-ecs:${BUILD_NUMBER} .'
                }
            }          
      stage("Push the docker image to ecr registry"){
            steps{
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'ecr_credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 421168438563.dkr.ecr.ap-south-1.amazonaws.com'
                    sh 'docker image push 421168438563.dkr.ecr.ap-south-1.amazonaws.com/spring-ecs:${BUILD_NUMBER}'
                }
            }
        }
        stage ('ecs-deploy') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'ecr_credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) 
                         {
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
    

