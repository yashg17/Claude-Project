pipeline {
    agent any

    environment {
        // Securely pulling your Claude API Key from Jenkins Credentials
        CLAUDE_API_KEY = credentials('CLAUDE_API_KEY')
    }

    stages {
        stage('Initialize & Sync') {
            steps {
                // 1. Download your latest code from Git
                checkout scm
                
                // 2. Fallback: Sync local files if they aren't in Git yet
                // (Matches your current local setup)
                sh 'cp /home/ubuntu/claude/*.py . || true'
                sh 'cp /home/ubuntu/claude/Dockerfile . || true'
                
                echo 'Environment prepared and files synced.'
            }
        }

        stage('Deploy App') {
            steps {
                sh '''
                    # Setup logging
                    touch app.log
                    chmod 666 app.log
                    
                    # Build and cycle container
                    docker build -t security-app .
                    docker stop my-app || true && docker rm my-app || true
                    
                    # Run with Environment Variable and Volume
                    docker run -d --name my-app \
                    -p 5000:5000 \
                    -e CLAUDE_API_KEY=${CLAUDE_API_KEY} \
                    -v "$(pwd)/app.log:/app/logs/app.log" \
                    security-app
                '''
            }
        }

        stage('Start AI Sentinel') {
            steps {
                sh '''
                    # Stop old sentinel processes
                    pkill -f security.py || true
                    
                    # Start the Python Sentinel in the background
                    nohup python3 -u security.py > sentinel.log 2>&1 &
                '''
            }
        }
    }
}
