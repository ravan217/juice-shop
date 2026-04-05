pipeline {
    agent any

    environment {
        APP_SERVER = 'deploy@10.114.0.3'
        APP_DIR = '/opt/juice-shop'
    }

    stages {

        stage('Checkout') {
            steps {
                sshagent(['target-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} \
                            "cd ${APP_DIR} && git pull origin main || \
                            git clone https://github.com/ravan217/juice-shop.git ${APP_DIR}"
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sshagent(['target-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} \
                            "cd ${APP_DIR} && npm install --legacy-peer-deps"
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sshagent(['target-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} \
                            "cd ${APP_DIR} && \
                            sed -i 's/condition =>/(condition: any) =>/g' lib/startup/validatePreconditions.ts && \
                            npm run build:server && \
                            cd frontend && npm install --legacy-peer-deps && npm run build"
                    '''
                }
            }
        }

        stage('Rebuild SQLite') {
            steps {
                sshagent(['target-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} \
                            "cd ${APP_DIR} && npm rebuild sqlite3"
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['target-server-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${APP_SERVER} \
                            "sudo systemctl restart juice-shop"
                    '''
                }
            }
        }

    }

    post {
        success {
            echo 'Deploy ugurla tamamlandi!'
        }
        failure {
            echo 'Xeta bas verdi!'
        }
    }
}
