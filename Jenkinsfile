pipeline {
    agent any // O un agente específico con Python y Bandit instalado

    environment {
        //CONSTANTES PARA BANDIT
        OUTPUT_PATH = "C:\\repogithub\\pygoat\\bandit_salida"
        
        //CONSTANTES PARA DEPENDENCY-Track
        // ID de la credencial configurada en Jenkins
        DEPENDENCY_TRACK_API_KEY = "odt_lxOz74Es_LaPQrq9ALXjw9e1VbhAxmshW6D77Z7Nj"
        DT_URL = "http://localhost:8081"
        PROJECT_ID = "e23cf72e-e768-4c10-94de-05d53fe43110"
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

        //Stage SAST de Bandit
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
        
        //Stage SCA de Dependency-Track
        stage('Dependency-Track Scan') {
            steps {
                // Publicar SBOM a Dependency-Track
                dependencyTrackPublisher(
                    //C:\\repogithub\\pygoat\\dependency_track_salida\\bom.xml
                    //
                    artifact: "${env.WORKSPACE}\\sbom.json", // Ruta al SBOM generado
                    synchronous: true, // Esperar resultados
                    projectId: "${env.PROJECT_ID}",
                    dependencyTrackUrl: "${env.DT_URL}",
                    dependencyTrackApiKey: "${env.DEPENDENCY_TRACK_API_KEY}"
                )
            }
        }
        
        stage('Gitleaks Scan') {
            steps {
                // Ejecuta gitleaks y genera el reporte html
                //bat 'C:\\Users\\HP\\AppData\\Local\\Microsoft\\WinGet\\Packages\\Gitleaks.Gitleaks_Microsoft.Winget.Source_8wekyb3d8bbwe\\gitleaks detect --source C:\\repogithub\\pygoat\\ --report-format html --report-path C:\\repogithub\\pygoat\\gitleaks_salida\\gitleaks-report.html'
                bat 'C:\\Users\\HP\\AppData\\Local\\Microsoft\\WinGet\\Packages\\Gitleaks.Gitleaks_Microsoft.Winget.Source_8wekyb3d8bbwe\\gitleaks detect --source C:\\repogithub\\pygoat\\ --verbose'
            }
        }
        
        //stage('Archive Report') {
        //    steps {
        //        // Guarda el reporte HTML como artefacto de la build
        //        archiveArtifacts artifacts: 'gitleaks-report.html', allowEmptyArchive: true
        //    }
        //}
        
        // Puedes añadir más etapas como DAST, Deploy, etc.
    }
    post {
        always {
            //archiveArtifacts artifacts: "${env.WORKSPACE}\\sbom.xml", onlyIfSuccessful: true
            // Limpieza opcional
            deleteDir()
        }
        failure {
            echo 'El pipeline falló.'
        }
    }
}