pipeline {
    agent any

    stages {
        stage('Build') { 
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            environment {
                npm_config_cache = './.npm-cache' // Avoid permission issue
            }
            steps {
                sh '''
                    echo "Listing files..."
                    ls -la

                    echo "Checking Node and npm versions..."
                    node --version
                    npm --version

                    echo "Installing dependencies..."
                    npm ci

                    echo "Building the app..."
                    npm run build

                    echo "Listing files after build..."
                    ls -la
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                    reuseNode true
                }
            }
            environment {
                PORT = "5000"
            }
            steps {
                sh '''
                    echo "Installing serve..."
                    npm install serve

                    echo "Starting server in the background..."
                    nohup node_modules/.bin/serve -s build -l $PORT > serve.log 2>&1 &

                    echo "Waiting for server to be available..."
                    for i in {1..10}; do
                        curl -s http://localhost:$PORT > /dev/null && break
                        echo "Waiting for server to start..."
                        sleep 2
                    done

                    echo "Running Playwright tests..."
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            script {
                if (fileExists('jest-results/junit.xml')) {
                    junit 'jest-results/junit.xml'
                } else {
                    echo 'No junit.xml file found to publish test results.'
                }
            }
        }
    }
}
