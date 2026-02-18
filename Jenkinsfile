pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/jlcesardev/ResCP-1.4.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                # Crear bucket producción si no existe
                aws s3 mb s3://production-todo-list-artifacts-622005079080 --region us-east-1 || true

                sam build

                sam deploy \
                    --stack-name production-todo-list-aws \
                    --s3-bucket production-todo-list-artifacts-622005079080 \
                    --capabilities CAPABILITY_IAM \
                    --region us-east-1 \
                    --parameter-overrides Stage=production \
                    --no-confirm-changeset \
                    --no-fail-on-empty-changeset
                '''
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
