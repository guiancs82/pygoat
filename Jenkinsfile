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

        stage('Analisis de Seguridad con Bandit') {
            steps {
                script {
                    // Define la imagen de Docker a usar (ej. python:3.9-slim)
                    def image = 'python:3.9-slim'
                    // Comando a ejecutar dentro del contenedor: instala Bandit y lo ejecuta sobre el código
                    // -v (volumen) monta el directorio actual del workspace a /app dentro del contenedor
                    // bandit -r /app -f json -o /tmp/bandit_report.json
                    def command = "pip install bandit && bandit -r C:\repogithub\pygoat -f json -o C:\repogithub\pygoat\bandit_salida\bandit_report.json"

                    // Ejecuta Docker y captura la salida estándar (stdout)
                    // La salida de este comando es el contenido del archivo JSON del reporte
                    def bandit_output = sh(
                        script: 'docker run --rm -v "${WORKSPACE}:/app" cytopia/bandit bandit /app" ${image} sh -c \"${command}\"',
                        returnStdout: true // Captura la salida del comando en la variable
                    ).trim() // Elimina espacios en blanco al inicio/final

                    // Opcional: Mostrar la salida en la consola de Jenkins
                    echo "Salida de Bandit (JSON): ${bandit_output}"

                    // Opcional: Si quieres el archivo, puedes copiarlo fuera del contenedor
                    // Nota: Asegúrate que el path /tmp/bandit_report.json existe en el contenedor
                    // Este paso ejecuta un comando adicional para copiar el archivo al workspace
                    sh """
                        docker run --rm -v "${workspace}:/app" ${image} sh -c "cp /tmp/bandit_report.json /app/bandit_report.json"
                    """
                    // Ahora puedes trabajar con el archivo bandit_report.json en tu workspace
                    echo "Archivo de reporte generado: ${workspace}/bandit_report.json"
                }
            }
        }
        // Puedes añadir una etapa para procesar los resultados, como un "quality gate"
        stage('Evaluar Resultados') {
            steps {
                script {
                    // Leer el archivo JSON y verificar si hay problemas (ej. criticidad alta)
                    // Aquí podrías usar una herramienta como 'jq' si está disponible en el agente
                    // Ejemplo: validar que no haya errores críticos
                    def critical_errors = sh(
                        script: "jq '.results[] | select(.level == \"ERROR\")' ${workspace}/bandit_report.json",
                        returnStdout: true
                    ).trim()

                    if (critical_errors) {
                        error "¡Bandit encontró errores críticos! Revisar ${workspace}/bandit_report.json"
                    } else {
                        echo "Análisis de Bandit completado sin errores críticos."
                    }
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