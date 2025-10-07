pipeline {
    agent any

    environment {
        NODE_VERSION = '20'
        HOST = 'http://localhost:3030'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Node.js') {
            steps {
                // Install Node.js 20 (requires NodeJS plugin in Jenkins)
                tool name: "node-${env.NODE_VERSION}", type: 'NodeJS'
                bat 'node --version' // or sh if on Linux agent
                bat 'npm --version'
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Update config.js') {
            steps {
                writeFile file: 'src/config.js', text: "export const settings = { host: '${HOST}' };"
            }
        }

        stage('Start services') {
            parallel {
                stage('Start Backend') {
                    steps {
                        bat 'start /B npm run start-be'
                    }
                }
                stage('Start Frontend') {
                    steps {
                        bat 'start /B npm run start-fe'
                    }
                }
            }
        }

        stage('Install Playwright browsers') {
            steps {
                bat 'npx playwright install'
            }
        }

        stage('Run Playwright tests') {
            steps {
                bat 'npm run test'
            }
        }

        stage('Deploy Backend') {
            when {
                branch 'main'
            }
            environment {
                RENDER_BE_SERVICE_ID = credentials('MY_RENDER_BE_SERVICE_ID')
                RENDER_API_KEY = credentials('MY_RENDER_API_KEY')
            }
            steps {
                sh '''
                    curl -X POST https://api.render.com/v1/services/$RENDER_BE_SERVICE_ID/deploys \
                    -H "Authorization: Bearer $RENDER_API_KEY"
                '''
            }
        }

        stage('Deploy Frontend') {
            when {
                branch 'main'
            }
            environment {
                RENDER_FE_SERVICE_ID = credentials('MY_RENDER_FE_SERVICE_ID')
                RENDER_API_KEY = credentials('MY_RENDER_API_KEY')
            }
            steps {
                sh '''
                    curl -X POST https://api.render.com/v1/services/$RENDER_FE_SERVICE_ID/deploys \
                    -H "Authorization: Bearer $RENDER_API_KEY"
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
        }
    }
}
