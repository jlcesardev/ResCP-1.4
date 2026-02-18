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
                sam build
                sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
        steps {
            sh '''
            . venv/bin/activate
            pip install pytest requests
            pytest
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
