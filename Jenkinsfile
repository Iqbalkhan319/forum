pipeline {
    agent any
    
    environment {
        APP_URL="http://forum.devops.com"
		APP_KEY="base64:EGCYLQuhy9xzmuPNnPvPNiXlpuiiOh0y4jI7N6/F5E0="
        CLIENT_APP_URL="http://forum.devops.com"
		APP_ENV="Production"
        APP_DEBUG="False"
		DB_HOST="172.16.188.136"
        DB_DATABASE="forumdb"
        DB_USERNAME="forum"
        DB_PASSWORD="'Ltest@12345'"

        DEPLOY_PATH = "/var/www/forum-pipe/"
		SSH_USER       = "deploy"
        DEPLOY_SERVER  = "172.16.188.137"
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from Git repository
                git branch: 'master', credentialsId: '9f1426ce-d7e3-4298-926b-b5cdd5501800', url: 'https://github.com/Iqbalkhan319/forum-pipe.git'
            }
        }
        stage('Change env') {
            steps {
                script {
                    sh '''
                    echo "Updating .env file with Jenkins environment variables..."
                    cp .env.example .env
                    sed -i "s|^DB_HOST=.*|DB_HOST=${DB_HOST}|" .env
                    sed -i "s|^DB_DATABASE=.*|DB_DATABASE=${DB_DATABASE}|" .env
                    sed -i "s|^DB_USERNAME=.*|DB_USERNAME=${DB_USERNAME}|" .env
                    sed -i "s|^DB_PASSWORD=.*|DB_PASSWORD=${DB_PASSWORD}|" .env
                    sed -i "s|^APP_URL=.*|APP_URL=${APP_URL}|" .env
                    sed -i "s|^CLIENT_APP_URL=.*|CLIENT_APP_URL=${CLIENT_APP_URL}|" .env
                    sed -i "s|^APP_DEBUG=.*|APP_DEBUG=${APP_DEBUG}|" .env
                    sed -i "s|^APP_KEY=.*|APP_KEY=${APP_KEY}|" .env
                    sed -i "s|^APP_ENV=.*|APP_ENV=${APP_ENV}|" .env
                    '''
                }
            }
        }
		stage('build') {
            steps {
                script {
                    sh '''
                    composer install --ignore-platform-reqs
                    '''
                }
            }
        }

        stage('PHPUnit Tests') {
            steps {
                // Run PHPUnit tests
                sh 'php artisan test'
            }
        }
        stage('Approval') {
            steps {
                input "Do you approve this build?"
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh '''
                ssh ${SSH_USER}@${DEPLOY_SERVER} "sudo mkdir -p '${DEPLOY_PATH}' && sudo chown -R deploy:deploy '${DEPLOY_PATH}'"

                rsync -avhP \
                    -e "ssh -o StrictHostKeyChecking=no" \
                    --exclude '.git/' \
                    ./ \
                    ${SSH_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/

                ssh ${SSH_USER}@${DEPLOY_SERVER} "
                    sudo chown -R www-data:www-data '${DEPLOY_PATH}'
                    cd '${DEPLOY_PATH}'
                    sudo -u www-data composer install --ignore-platform-reqs
                    sudo -u www-data php artisan config:clear
                    sudo -u www-data php artisan cache:clear
                    sudo -u www-data php artisan migrate
                "
            '''
                }
            }
        }

    }

    post {
        success {
            echo 'UAT deployment successful!'
            mail to: 'iqbalkhan319@gmail.com',
                 from: 'buprojecttime@brac.net',
                 subject: "UAT Pipeline Success: ${currentBuild.fullDisplayName}",
                 body: "UAT deployment successful for build ${env.BUILD_NUMBER}.\nView console output at ${env.BUILD_URL}"
        }

        failure {
            echo 'UAT deployment failed!'
            emailext attachLog: true,  // Attach the build log to the email
                    to: 'iqbalkhan319@gmail.com',
                    from: 'buprojecttime@brac.net',
                    subject: "UAT Pipeline Failure: ${currentBuild.fullDisplayName}",
                    body: "UAT deployment failed for build ${env.BUILD_NUMBER}.\nView console output at ${env.BUILD_URL}"
        }
        always {
            // Clean up after the pipeline run
            cleanWs()
        }
    }
}
