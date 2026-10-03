pipeline {
    agent any

    tools {
        nodejs 'Node_24'
    }

    environment {
        SONAR_PROJECT_KEY = 'ucp-app-react'
        SONAR_PROJECT_NAME = 'UCP React App'
    }

    stages {
        // Etapa 1: Checkout del código desde GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Tatianag2/ucp-app-react.git'
            }
        }

        // Etapa 2: Instalar dependencias y build del proyecto
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        // Etapa 3: Pruebas Unitarias + Cobertura (JUnit + LCOV)
        stage('Pruebas Unitarias') {
            steps {
                sh 'npm test -- --watchAll=false --ci --coverage --reporters=default --reporters=jest-junit'
            }
            post {
                always {
                    junit 'junit.xml'
                    archiveArtifacts artifacts: 'junit.xml', allowEmptyArchive: true
                }
            }
        }

        // Etapa 4: Análisis estático de código en SonarQube
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                              -Dsonar.sources=src \
                              -Dsonar.host.url=http://sonarqube:9000 \
                              -Dsonar.login=\${SONAR_AUTH_TOKEN} \
                              -Dsonar.javascript.node=\${NODEJS_HOME}/bin/node \
                              -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                        """
                    }
                }
            }
        }

        // Etapa 5: Esperar aprobación del Quality Gate de SonarQube
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline abortado: Quality Gate no aprobado (${qg.status})"
                        }
                    }
                }
            }
        }
    }

    // Post-actions: Notificación por correo con estado final
    post {
        always {
            emailext (
                subject: "Pipeline ${currentBuild.result}: ucp-app-react #${env.BUILD_NUMBER}",
                body: """
                    Estado del Build: ${currentBuild.result}
                    URL del Build: ${env.BUILD_URL}
                    Detalles de Pruebas: ${env.BUILD_URL}testReport/
                    SonarQube: http://localhost:9000/dashboard?id=${SONAR_PROJECT_KEY}
                """,
                to: 'pruebaggtggv@gmail.com'
            )
        }
    }
}