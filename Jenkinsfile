pipeline {
    agent any // O un agente específico con Python y Bandit instalado

    environment {
        //CONSTANTES PARA BANDIT
        OUTPUT_PATH = "C:\\repogithub\\pygoat\\bandit_salida"
        
        //CONSTANTES DEL DEFECT DOJO
        PRODUCT_ID = "1"
        ENGAGEMENT_ID = "1"
        DOJO_URL = "http://localhost:8084"
        DOJO_TOKEN = "2d99e68e27e24a429cdf3e697e0df08ce24e575f"
        
        //CONSTANTES PARA DEPENDENCY-Track
        // ID de la credencial configurada en Jenkins
        DEPENDENCY_TRACK_API_KEY = "odt_lxOz74Es_LaPQrq9ALXjw9e1VbhAxmshW6D77Z7Nj"
        DT_URL = "http://localhost:8081"
        PROJECT_ID = "e23cf72e-e768-4c10-94de-05d53fe43110"
        SBOM_FILE = "C:\\repogithub\\pygoat\\dependency_track_salida\\sbom.json"
        WORKSPACE = "C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\proyecto_final_pygoat\\"
    }

    stages {
        
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
                    bat "C:\\Python314\\Scripts\\bandit.exe -r . -f json -o ${OUTPUT_PATH}\\reporte.json --exit-zero"
                    bat "C:\\Python314\\Scripts\\bandit.exe -r . -f html -o ${OUTPUT_PATH}\\reporte.html --exit-zero"
                }
            }
        }
        
        //Stage para integrar defect dojo consumiento la salida de bandit
        stage('Upload to DefectDojo') {
            steps {
                // Usando el plugin oficial de DefectDojo
                defectDojoPublisher(
                    artifact: "${env.OUTPUT_PATH}\\reporte.json",
                    scanType: "Bandit Scan",
                    //defectDojoUrl: "${env.DOJO_URL}",
                    //defectDojoCredentialsId: "${env.DOJO_TOKEN}",
                    productId: "${env.PRODUCT_ID}",
                    engagementId: "${env.ENGAGEMENT_ID}"
                    //engagementId: "${env.ENGAGEMENT_ID}",
                    //autoCreateEngagements: false,
                    //autoCreateProducts: false
                )
            }
        }
        
        //Stage SCA de Dependency-Track
        stage('Dependency-Track Scan') {
            steps {
                // Publicar SBOM a Dependency-Track
                dependencyTrackPublisher(
                    artifact: "${env.WORKSPACE}\\sbom.json", // Ruta al SBOM generado
                    synchronous: true, // Esperar resultados
                    projectId: "${env.PROJECT_ID}",
                    dependencyTrackUrl: "${env.DT_URL}",
                    dependencyTrackApiKey: "${env.DEPENDENCY_TRACK_API_KEY}"
                )
            }
        }
        
        
        //Stage SAST de Bandit con vulnerabilidades altas o superior
        stage('SAST Scan with Bandit vulnerabilidades altas') {
            steps {
                script {
                    // Crea la carpeta de salida si no existe
                    bat "if not exist ${OUTPUT_PATH} mkdir ${OUTPUT_PATH}"
                    
                    // Ejecuta Bandit y guarda la salida en archivos
                    // -r: recursivo, -f: formato, -o: archivo de salida
                    // Se usa '|| exit 0' para que el pipeline no falle si encuentra vulnerabilidades (opcional)
                    bat "C:\\Python314\\Scripts\\bandit.exe -r . -f html -o ${OUTPUT_PATH}\\reporteHighVul.html -ll"
                }
            }
        }
        
        
        //Stage para analizar secretos con Gitleaks
        stage('Gitleaks Scan') {
            steps {
                // Ejecuta gitleaks y genera el reporte html
                bat 'C:\\Users\\HP\\AppData\\Local\\Microsoft\\WinGet\\Packages\\Gitleaks.Gitleaks_Microsoft.Winget.Source_8wekyb3d8bbwe\\gitleaks detect --source . --verbose'
            }
        }        
        
        //Stage SCA de Dependency-Track con punto de parada si  hay una vulnerabilidad alta
        stage('Dependency-Track Scan con punto de parada Alta') {
            steps {
                // Publicar SBOM a Dependency-Track
                dependencyTrackPublisher(
                    //
                    artifact: "${env.WORKSPACE}\\sbom.json", // Ruta al SBOM generado
                    synchronous: true, // Esperar resultados
                    projectId: "${env.PROJECT_ID}",
                    dependencyTrackUrl: "${env.DT_URL}",
                    dependencyTrackApiKey: "${env.DEPENDENCY_TRACK_API_KEY}",
                    failedTotalHigh: 0     // Falla si hay 1 alta
                )
            }
        }
        
    }
    post {
        always {
            // Limpieza opcional
            deleteDir()
        }
        failure {
            echo 'El pipeline falló.'
        }
    }
}