pipeline {
    agent any // O un agente específico con Python y Bandit instalado

    environment {
        // Define el nombre del reporte
        REPORT_NAME = "bandit-report.json"
    }

    stages {
        stage('Checkout') {
            steps {
                // Clona el código del repositorio
                git url: 'https://github.com/guiancs82/pygoat', branch: 'main' // O la rama que necesites
            }
        }

        stage('Build (Opcional)') {
            steps {
                // Si tu proyecto necesita ser construido (ej: pip install -r requirements.txt)
                sh 'pip install -r requirements.txt'
                echo 'Proyecto construido (si es necesario)'
            }
        }

        stage('SAST con Bandit') {
            steps {
                script {
                    // Ejecuta Bandit mediante la imagen oficial de PyCQA en Docker
                    // Se monta el directorio actual (%WORKSPACE% en Windows) al contenedor
                    bat """
                    docker run --rm -v "%WORKSPACE%":/app cytopia/bandit -r /code -f json -o /code/${REPORT_NAME} || exit 0
                    """
                }
            }
        }
        // Puedes añadir más etapas como DAST, Deploy, etc.
    }
    post {
        always {
            // Limpieza opcional
            deleteDir()
        }
        failure {
            echo 'El pipeline falló en la etapa SAST.'
        }
    }
}