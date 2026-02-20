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

                    ...
                }
            }
        }

        stage('Deploy') {
            steps {
                script {

                    echo "Deploying using environment: ${configBranch}"

                    sh """
                        sam build
                        sam deploy \
                          --template-file .aws-sam/build/template.yaml \
                          --config-file samconfig.toml \
                          --config-env ${configBranch} \
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

            sh """
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install pytest requests

                export BASE_URL=${baseUrl}

                pytest test/integration/todoApiTest.py -k "get or list" -v
            """
        }
		}
	}
			
    }
}
