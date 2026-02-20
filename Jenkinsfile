pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {


        stage('Get Code') {
         agent any
         steps {
         script {

            // Detectar rama que dispara el pipeline
            def appBranch = env.BRANCH_NAME ?: "master"
            def configBranch = (appBranch == "master") ? "production" : "staging"

            echo "Application branch: ${appBranch}"
            echo "Config branch: ${configBranch}"

            // Clonar aplicación
            git branch: appBranch,
                url: 'https://github.com/jlcesardev/ResCP-1.4.git'

            // Clonar repo de configuración según entorno
            sh """
		   rm -rf config
		   git clone -b ${configBranch} https://github.com/jlcesardev/todo-list-aws-config.git config
		   cp config/samconfig.toml .
		   echo "samconfig.toml descargado:"
		   cat samconfig.toml
	        """
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
        sh '''
        python3 -m venv venv
        . venv/bin/activate
        pip install --upgrade pip
        pip install pytest requests

        BASE_URL="https://ky2falsixf.execute-api.us-east-1.amazonaws.com/Prod"
        export BASE_URL=$BASE_URL

        pytest test/integration/todoApiTest.py -k "get or list" -v
        '''
    }
}



   }
}
