pipeline {
    agent any // O un agente específico con Python y Bandit instalado

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
                // Instala Bandit si no está en el agente
                sh 'pip install bandit'
                // Ejecuta Bandit en el código fuente (ejemplo para Python)
                sh 'bandit -r . -f json -o bandit-results.json'
                echo 'Escaneo SAST con Bandit completado.'
                // Opcional: Publicar resultados como artefacto
                archiveArtifacts artifacts: 'bandit-results.json', fingerprint: true
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