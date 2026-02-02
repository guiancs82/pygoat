pipeline {
    agent any // O un agente específico con Python y Bandit instalado

    environment {
        //CONSTANTES PARA BANDIT
        OUTPUT_PATH = "C:\\repogithub\\pygoat\\bandit_salida"
        
        //CONSTANTES PARA DEPENDENCY-Track
        // ID de la credencial configurada en Jenkins
        DEPENDENCY_TRACK_API_KEY = credentials('odt_mpVZfuV1_CkTRxoyvW2d8DveOG8kmpTchOhc6F4Lj')
        DT_URL = 'http://localhost:8082'
        PROJECT_ID = 'PYGOAT'
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
        stage('Build & SBOM') {
            steps {
                // Generar SBOM usando Maven (o herramientas como cdxgen)
                sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
            }
        }
        stage('Dependency-Track Scan') {
            steps {
                // Publicar SBOM a Dependency-Track
                dependencyTrackPublisher(
                    artifact: '**/target/dependency-track/bom.xml', // Ruta al SBOM generado
                    synchronous: true, // Esperar resultados
                    projectId: "${env.PROJECT_ID}",
                    dependencyTrackUrl: "${env.DT_URL}",
                    apiToken: "${env.DEPENDENCY_TRACK_API_KEY}"
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