// jenkinsfile for pipeline

pipeline {
	agent any
	environment {
		DOCKERHUB_CREDS = credentials('docker') 
	}
	stages {

		stage('Docker Image Build') {
			steps {
				echo 'Building Docker Image...'
				sh 'docker build --tag lander302/cw2-again-server:1.0 .'
				echo 'Docker Image built successfully!'
			}
		}
		
		stage('Test Docker Image') {
			steps {
				echo 'Testing Docker Image...'
				sh '''
					docker image inspect lander302/cw2-again-server:1.0
					docker run --name test-container -p 8081:8080 -d lander302/cw2-again-server:1.0
					docker ps
					docker stop test-container
					docker rm test-container
				'''
			}
		}
	
		stage('DockerHub Login') {
			steps {
				sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
			}
		}
		
		stage('DockerHub Image Push') {
			steps {
				sh 'docker push lander302/cw2-again-server:1.0'
			}
		}
		
		stage('Deploy') {
			steps {
				sshagent(['jenkins-k8s-ssh-key']) {
					sh '''
						ssh -o StrictHostKeyChecking=no ubuntu@ec2-54-167-153-197.compute-1.amazonaws.com \
						"export PATH=$PATH:/usr/bin && ansible-playbook /home/ubuntu/ansible/deployToKubernetes.yml --extra-vars 'dockerhub_image=lander302/cw2-server:1.0'"
					'''	
				}
			}		
		}
	}
}
