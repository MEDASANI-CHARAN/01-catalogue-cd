pipeline {
	agent {
        label 'AGENT-1'
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '009160060207'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }
    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the environment')
    }
	stages {
		stage('Docker Build'){
			steps {
				script {
                        withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                            sh """
                                aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                                kubectl get nodes
                                kubectl apply -f namespace.yaml
                                sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                                helm upgrade --install ${COMPONENT} -f values-${params.deploy_to}.yaml -n ${PROJECT} .
                            """
                        }
                }
			}
		}
        stage('Check status'){
			steps {
				script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        // Execute a shell command and capture its standard output
                    def deploymentStatus = sh(returnStdout: true, script: 'kubectl rollout status deployment/catalogue-deployment --timeout=30s -n ${PROJECT} || echo FAILED').trim()

                    // Print the captured output to the Jenkins console
                    echo "Output of 'kubectl rollout status deployment/catalogue-deployment --request-timeout=30s':"
                    echo deploymentStatus

                    // You can also use the output in further steps
                    if (deploymentStatus.contains("successfully rolled out")) {
                        echo "Deployment is success!"
                    } else {
                        sh"""
                            helm rollback ${COMPONENT} -n ${PROJECT}
                            sleep 20
                        """
                        def rollbackStatus = sh(returnStdout: true, script: 'kubectl rollout status deployment/catalogue-deployment --timeout=30s || echo FAILED').trim()
                        if (rollbackStatus.contains("successfully rolled out")) {
                        error "Deployment is failure, Rollback is success"
                        }
                        else {
                            error "Deployment is failure, Rollback is failure. Application is not running"
                            }
                        }
                    }                    
                }
            }
        }
    }
	
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'I will always say success!'
        }
        failure { 
            echo 'I will always say failure!'
        }
    }
}