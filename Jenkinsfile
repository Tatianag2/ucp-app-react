pipeline {
    agent any

    tools {
        nodejs 'Node_24'
        sonarScanner 'SonarQubeScanner'
    }

    environment {
        SONAR_PROJECT_KEY = 'ucp-app-react'
        SONAR_PROJECT_NAME = 'UCP React App'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Tatianag2/ucp-app-react.git'
            }
        }

        stage('Build & Test Coverage') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                sh 'npm run test:coverage'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                          -Dsonar.sources=src \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=${SONAR_AUTH_TOKEN} \
                          -Dsonar.javascript.node=${NODEJS_HOME}/bin/node \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Calidad no aprobada: ${qg.status}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo '¡Pipeline ejecutado con éxito y Quality Gate aprobado!'
        }
        failure {
            echo 'Pipeline fallido. Revisar logs.'
        }
    }
}