pipeline {
    agent any

    environment {
        SONAR_HOME = tool "sonar-scanner"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/AbhishekAbhisharma/three-tier-app.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                    \$SONAR_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=three-tier-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://host.docker.internal:9000 \
                    -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t 59005/frontend:latest ./frontend'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t 59005/backend:latest ./backend'
            }
        }

        stage('Trivy Scan Frontend') {
            steps {
                sh 'trivy image 59005/frontend:latest'
            }
        }

        stage('Trivy Scan Backend') {
            steps {
                sh 'trivy image 59005/backend:latest'
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh 'docker push 59005/frontend:latest'
            }
        }

        stage('Push Backend Image') {
            steps {
                sh 'docker push 59005/backend:latest'
            }
        }

    }
}
