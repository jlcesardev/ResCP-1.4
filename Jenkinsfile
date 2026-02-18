pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'eu-west-1'
    }

    stages {

        stage('Static Test') {
        steps {
            sh '''
            python3 -m venv venv
            . venv/bin/activate
            pip install flake8 bandit
            flake8 src || true
            bandit -r src || true
            '''
    }
}


        stage('Deploy') {
        steps {
        sh '''
        # Crear bucket si no existe (no falla si ya existe)
        aws s3 mb s3://staging-todo-list-artifacts-622005079080 --region us-east-1 || true

        # Build SAM
        sam build

        # Deploy SAM no interactivo
        sam deploy \
            --stack-name staging-todo-list-aws \
            --s3-bucket staging-todo-list-artifacts-622005079080 \
            --capabilities CAPABILITY_IAM \
            --region us-east-1 \
            --parameter-overrides Stage=staging \
            --no-confirm-changeset \
            --no-fail-on-empty-changeset
        '''
    }
}

       stage('Rest Test') {
    steps {
        sh '''
        echo "Activating virtual environment..."
        . venv/bin/activate

        echo "Installing test dependencies..."
        pip install pytest requests

        echo "Running pytest..."
        pytest tests/ -v --maxfail=1 --disable-warnings
        '''
    }
}


        stage('Promote') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GIT_TOKEN')]) {
                    sh '''
                    git config user.email "ci@jenkins.local"
                    git config user.name "Jenkins CI"
                    git checkout master
                    git merge develop
                    git push https://${GIT_TOKEN}@github.com/TU_USUARIO/todo-list-aws.git master
                    '''
                }
            }
        }
    }
}
