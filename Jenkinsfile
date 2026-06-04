pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', defaultValue: 'latest', description: 'Tag de la imagen Docker')
        booleanParam(name: 'DO_PUSH', defaultValue: false, description: 'Publicar imagen en DockerHub')
    }

    environment {
        DOCKERHUB_USER = 'juliotechsys'
        IMAGE_NAME = 'ceste-ci-demo'
        FULL_IMAGE = "${DOCKERHUB_USER}/${IMAGE_NAME}:${params.APP_VERSION}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Build application') {
            steps {
                bat 'npm run build'
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
            }
        }

        stage('SonarQube analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        bat "\"${scannerHome}/bin/sonar-scanner.bat\""
                    }
                }
            }
        }

        stage('Docker build') {
            steps {
                bat 'docker build -t %FULL_IMAGE% .'
            }
        }

        stage('Trivy image scan') {
            steps {
                bat 'if not exist reports mkdir reports'
                bat 'trivy image --severity HIGH,CRITICAL --format table --output reports\\trivy-report.txt %FULL_IMAGE%'
                archiveArtifacts artifacts: 'reports/trivy-report.txt', allowEmptyArchive: true
                bat 'type reports\\trivy-report.txt'
            }
        }

        stage('DockerHub push') {
            when {
                expression { return params.DO_PUSH }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                        docker login -u "%DOCKER_USER%" -p "%DOCKER_PASS%"
                        docker push "%FULL_IMAGE%"
                    '''
                }
            }
        }
    }

    post {
        always {
            bat 'docker logout || exit /b 0'
        }
    }
}