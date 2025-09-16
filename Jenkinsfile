pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "muggulla6/jenkinstestdemo"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh '. venv/bin/activate && pytest tests/ --maxfail=1 --disable-warnings -q'
            }
        }

        stage('Build & Push Docker Image') {
            when {
                branch 'main'   // ✅ only run for main branch
            }
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${env.BRANCH_NAME} ."
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${env.BRANCH_NAME}"
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'   // ✅ only deploy from main branch
            }
            steps {
                sh """
                    docker stop jenkinstestdemo || true
                    docker rm jenkinstestdemo || true
                    docker run -d --name jenkinstestdemo -p 8081:8080 ${DOCKER_IMAGE}:${env.BRANCH_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Build for branch ${env.BRANCH_NAME} succeeded!"
        }
        failure {
            echo "❌ Build for branch ${env.BRANCH_NAME} failed!"
        }
    }
}
