pipeline {
	agent any
	stages {

				// stage('Create kubernetes cluster') {
				// 	steps {
				// 		withAWS(region:'us-west-2', credentials:'aws-creds') {
				// 			sh '''
				// 				eksctl create cluster \
				// 				--name capstone-project \
				// 				--version 1.13 \
				// 				--nodegroup-name standard-workers \
				// 				--node-type t2.small \
				// 				--nodes 2 \
				// 				--nodes-min 1 \
				// 				--nodes-max 3 \
				// 				--node-ami auto \
				// 				--region us-west-2 \
				// 				--zones us-west-2a \
				// 				--zones us-west-2b \
				// 				--zones us-west-2c \
				// 			'''
				// 		}
				// 	}
				// }

		

		// stage('Create config file cluster') {
		// 	steps {
		// 		withAWS(region:'us-west-2', credentials:'aws-creds') {
		// 			sh '''
		// 				aws eks --region us-west-2 update-kubeconfig --name capstone-project
		// 			'''
		// 		}
		// 	}
		// }

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t atharva11/udacity-capstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push atharva11/udacity-capstone
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-creds') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:134748528531:cluster/capstone-project
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-creds') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-creds') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-creds') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-creds') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}


		
	}
}