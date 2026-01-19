pipeline {
    agent any // O un agente específico con Python y Bandit instalado

    environment {
        // Define el nombre del reporte
        OUTPUT_PATH = "C:\\repogithub\\pygoat\\bandit_salida"
    }

    stages {
        stage('Checkout') {
            steps {
                // Clona el código del repositorio
                git url: 'https://github.com/guiancs82/pygoat.git', branch: 'main' // O la rama que necesites
            }
        }

        stage('SAST Scan with Bandit') {
            steps {
                script {
                    // Crea la carpeta de salida si no existe
                    bat "if not exist ${OUTPUT_PATH} mkdir ${OUTPUT_PATH}"
                    
                    // Ejecuta Bandit y guarda la salida en archivos
                    // -r: recursivo, -f: formato, -o: archivo de salida
                    // Se usa '|| exit 0' para que el pipeline no falle si encuentra vulnerabilidades (opcional)
                    bat "C:\\Users\\HP\\AppData\\Roaming\\Python\\Python314\\Scripts\\bandit.exe -r . -f json -o ${OUTPUT_PATH}\\reporte.json --exit-zero"
                    bat "C:\\Users\\HP\\AppData\\Roaming\\Python\\Python314\\Scripts\\bandit.exe -r . -f html -o ${OUTPUT_PATH}\\reporte.html --exit-zero"
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