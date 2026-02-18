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

            def appBranch = env.BRANCH_NAME ?: "master"
	    def configEnv = (appBranch == "master") ? "production" : "staging"

            echo "Deploying with config-env: ${configEnv}"

            sh """
                sam build
                sam deploy --config-env ${configEnv}
            """
        }
    }
}



        stage('Rest Test') {
            steps {
                sh '''
                echo "Setting BASE_URL production..."

                BASE_URL="https://t7smiakg40.execute-api.us-east-1.amazonaws.com/Prod"
                export BASE_URL=$BASE_URL

                echo "BASE_URL: $BASE_URL"

                # Ejecutar SOLO tests de lectura
                venv/bin/pytest test/integration/todoApiTest.py -k "get or list" -v
                '''
            }
        }

    }
}
