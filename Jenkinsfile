pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                script {

                    def appBranch = env.BRANCH_NAME ?: "master"
                    configBranch = (appBranch == "master") ? "production" : "staging"

                    echo "Application branch: ${appBranch}"
                    echo "Config branch: ${configBranch}"

                
                }
            }
        }


		stage('Deploy') {
    steps {
        script {

            def appBranch = env.BRANCH_NAME
            def configBranch = (appBranch == "master") ? "production" : "staging"
            def stackName = (appBranch == "master") ? "production-todo-list-aws" : "staging-todo-list-aws"

            echo "Application branch: ${appBranch}"
            echo "Deploying using config: ${configBranch}"
            echo "Stack name: ${stackName}"

           sh """
    		sam build
    		sam deploy \
      		--template-file .aws-sam/build/template.yaml \
      		--stack-name ${stackName} \
      		--capabilities CAPABILITY_IAM \
      		--region us-east-1 \
      		--resolve-s3 \
      		--no-confirm-changeset \
      		--no-fail-on-empty-changeset
			"""
        }
    }
}

		
		
		stage('Rest Test') {
    	steps {
        script {

            def baseUrl = (configBranch == "production") ?
                "https://ky2falsixf.execute-api.us-east-1.amazonaws.com/Prod" :
                "https://TU_URL_STAGING.execute-api.us-east-1.amazonaws.com/Prod"

	
echo "Base URL detected: ${baseUrl}"

sh """
    export BASE_URL=${baseUrl}
    pytest test/integration/todoApiTest.py -k "get or list" -v
"""

			
        }
		}
	}
			
    }
}
