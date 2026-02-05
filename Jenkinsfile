pipeline {
    agent any // O un agente específico con Python y Bandit instalado

    environment {
        //CONSTANTES PARA BANDIT
        OUTPUT_PATH = "C:\\repogithub\\pygoat\\bandit_salida"
        
        //CONSTANTES PARA DEPENDENCY-Track
        // ID de la credencial configurada en Jenkins
        DEPENDENCY_TRACK_API_KEY = "odt_lxOz74Es_LaPQrq9ALXjw9e1VbhAxmshW6D77Z7Nj"
        DT_URL = "http://localhost:8081"
        PROJECT_ID = "PYGOAT"
        SBOM_FILE = "/opt/owasp/dependency-track/dependency_track_salida/bom.xml"
        WORKSPACE = "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\proyecto_final_pygoat\\"
    }

    stages {
        //stage('Cleanup') {
        //    steps {
        //        node { // or 'agent any' above
        //            deleteDir()
        //        }
        //   }
        //}
    
        //Baja una copia del repositorio de pygoat
        stage('Checkout') {
            steps {
                // Clona el código del repositorio
                //git url: 'https://github.com/guiancs82/pygoat.git', branch: 'main' // O la rama que necesites
                checkout scm
            }
        }

        //Stage de Bandit
        stage('SAST Scan with Bandit') {
            steps {
                script {
                    // Crea la carpeta de salida si no existe
                    bat "if not exist ${OUTPUT_PATH} mkdir ${OUTPUT_PATH}"
                    
                    // Ejecuta Bandit y guarda la salida en archivos
                    // -r: recursivo, -f: formato, -o: archivo de salida
                    // Se usa '|| exit 0' para que el pipeline no falle si encuentra vulnerabilidades (opcional)
                    //C:\\Users\\HP\\AppData\\Roaming\\Python\\Python314\\Scripts\\
                    bat "C:\\Python314\\Scripts\\bandit.exe -r . -f json -o ${OUTPUT_PATH}\\reporte.json --exit-zero"
                    bat "C:\\Python314\\Scripts\\bandit.exe -r . -f html -o ${OUTPUT_PATH}\\reporte.html --exit-zero"
                }
            }
        }
        
        //Stage de Dependency-Track
        //stage('Build & SBOM') {
        //    steps {
                // Generar SBOM usando Maven (o herramientas como cdxgen)
                //bat 'npm init -y'
        //        bat 'npm install -g @cyclonedx/cyclonedx-npm'
        //        bat 'set PATH=%PATH%;C:\\Users\\HP\\AppData\\Roaming\\npm'
        //        bat 'C:\\Users\\HP\\AppData\\Roaming\\npm\\cyclonedx-npm --output-file C:\\repogithub\\pygoat\\dependency_track_salida\\sbom.json'
        //    }
        //}
        stage('Dependency-Track Scan') {
            steps {
                // Publicar SBOM a Dependency-Track
                dependencyTrackPublisher(
                    //C:\\repogithub\\pygoat\\dependency_track_salida\\bom.xml
                    //
                    artifact: "${env.WORKSPACE}\\sbom.json", // Ruta al SBOM generado
                    synchronous: true, // Esperar resultados
                    projectId: "${env.PROJECT_ID}",
                    dependencyTrackUrl: "${env.DT_URL}"
                    //,synchronous: true,
                    //apiKey: "${env.DEPENDENCY_TRACK_API_KEY}"
                )
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