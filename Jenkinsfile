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
                                helm upgrade --install ${COMPONENT} -f values-${params.deploy_to}.yaml -n ${PROJECT}.
                            """
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