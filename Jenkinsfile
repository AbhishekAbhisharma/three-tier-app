pipeline {
    agent any

    environment {
        SONAR_HOME = tool "sonar-scanner"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AbhishekAbhisharma/three-tier-app.git'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }

            steps {
                withSonarQubeEnv('sonar-server') {

                    sh '''
                    $SONAR_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=three-tier-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://host.docker.internal:9000 \
                    -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t abhisheksharma9877/frontend:latest ./frontend'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t abhisheksharma9877/backend:latest ./backend'
            }
        }

        stage('Trivy Scan Frontend') {
            steps {
                sh 'trivy image abhisheksharma9877/frontend:latest'
            }
        }

        stage('Trivy Scan Backend') {
            steps {
                sh 'trivy image abhisheksharma9877/backend:latest'
            }
        }

        stage('Docker Login') {
            environment {
                DOCKER_CREDS = credentials('dockerhub-creds')
            }

            steps {
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh 'docker push abhisheksharma9877/frontend:latest'
            }
        }

        stage('Push Backend Image') {
            steps {
                sh 'docker push abhisheksharma9877/backend:latest'
            }
        }
    }
}
