pipeline {
    agent any

    tools {
        nodejs 'Node_24' // Nombre definido en Global Tool Configuration
    }

    stages {
        // Etapa 1: Checkout del código desde GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/amartinezh/ucp-app-react.git'
            }
        }

        // Etapa 2: Instalar dependencias y build del proyecto
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        // Etapa 3: Ejecutar pruebas unitarias
        stage('Unit Tests') {
            steps {
                sh 'npm test -- --watchAll=false --silent > test-output.txt || true'
                sh 'cat test-output.txt'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'test-output.txt', allowEmptyArchive: true
                }
            }
        }
    }

    post {
        success {
            echo '¡Pipeline ejecutado con éxito!'
        }
        failure {
            echo 'Pipeline fallido. Revisar logs.'
        }
    }
}