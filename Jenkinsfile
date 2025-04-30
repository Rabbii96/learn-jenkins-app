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
                npm_config_cache = './.npm-cache'
            }
            steps {
                sh '''
                    echo "Checking Node and npm versions..."
                    node --version
                    npm --version

                    echo "Installing dependencies..."
                    npm ci

                    echo "Building the app..."
                    npm run build
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
                    if [ ! -d "build" ]; then
                        echo "Build folder not found. Exiting."
                        exit 1
                    fi

                    echo "Installing serve..."
                    npm install serve

                    echo "Starting server in background..."
                    nohup npx serve -s build -l $PORT > serve.log 2>&1 &

                    echo "Waiting for server to be available..."
                    for i in {1..15}; do
                        curl -s http://localhost:$PORT > /dev/null && break
                        echo "Still waiting for server ($i)..."
                        sleep 2
                    done

                    echo "Running Playwright tests..."
                    npx playwright test || (echo "Playwright test failed. Printing server logs:" && cat serve.log && exit 1)
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
                    echo 'No junit.xml file found.'
                }

                // Print serve logs if available
                if (fileExists('serve.log')) {
                    echo '--- Serve Log ---'
                    def log = readFile('serve.log')
                    echo log
                }
            }
        }
    }
}
