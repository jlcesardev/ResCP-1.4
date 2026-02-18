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

            // Extraer el nombre del stack desde samconfig
            def stackName = sh(
                script: "grep stack_name samconfig.toml | cut -d '\"' -f2",
                returnStdout: true
            ).trim()

            echo "Deploying stack: ${stackName}"

		sh """
		    sam build
		"""
		sh """
		    sam deploy \
		      --template-file .aws-sam/build/template.yaml \
		      --stack-name ${stackName} \
		      --capabilities CAPABILITY_IAM \
		      --region us-east-1 \
		      --resolve-s3 \
		      --no-confirm-changeset
		"""
	   }
    }
}

	stage('Rest Test') {
	    steps {
        sh '''
        echo "Setting BASE_URL production..."

        BASE_URL="https://ky2falsixf.execute-api.us-east-1.amazonaws.com/Prod"
        export BASE_URL=$BASE_URL

        echo "BASE_URL: $BASE_URL"

        python3 -m venv venv
        . venv/bin/activate
        pip install --upgrade pip
        pip install pytest requests

        pytest test/integration/todoApiTest.py -k "get or list" -v
        '''
    	}
	}
   }
}
