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
        echo "Setting BASE_URL..."

        BASE_URL="https://t7smiakg40.execute-api.us-east-1.amazonaws.com/Prod"
        export BASE_URL=$BASE_URL

        echo "BASE_URL: $BASE_URL"

        venv/bin/pytest test/integration/todoApiTest.py -v
        '''
    }
}

    stage('Promote') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'github-token',
            usernameVariable: 'GIT_USER',
            passwordVariable: 'GIT_PASS'
        )]) {
            sh '''
            git config user.email "ci@jenkins.local"
            git config user.name "Jenkins CI"

            git fetch origin

            git checkout master
            git merge origin/develop

            git push https://${GIT_USER}:${GIT_PASS}@github.com/jlcesardev/ResCP-1.4.git master
            '''
        }
    }
}



    }
}
