// Jenkinsfile para la construcción y despliegue de Spring5 en Tomcat

pipeline {
    // Define el entorno donde se ejecutará el Pipeline (ej. en cualquier agente disponible)
    agent any

    // Define los parámetros si los necesitas (opcional)
    parameters {
        string(name: 'TOMCAT_APP_NAME', defaultValue: 'Spring5', description: 'Nombre de la aplicación en Tomcat (context path)')
    }

    // Etapas del proceso
    stages {
        stage('Checkout') {
            steps {
                echo 'Obteniendo código del repositorio...'
                // Por defecto, Jenkins obtiene el código configurado en la definición del Job.
                // Si quieres ser explícito, puedes usar:
                // git url: 'https://github.com/labdesarrollojava/Spring5.git', branch: 'main'
            }
        }

        stage('Build WAR') {
            steps {
                echo 'Compilando y empaquetando el proyecto con Maven...'
                // Asume que tienes Maven configurado en Jenkins (en Manage Jenkins > Tools)
                // Usamos 'sh' para ejecutar un comando de shell.
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Iniciando el despliegue en Tomcat...'
                // Esta es la acción que requiere el plugin "Deploy to Container Plugin"
                deploy adapters: [tomcat8(
                    // ID de las credenciales de Tomcat que configuraste en Jenkins
                    credentialsId: '<TU_ID_CREDENCIALES_TOMCAT>',
                    // URL base de tu servidor Tomcat (ej. http://localhost:8080)
                    url: '<URL_TOMCAT>' 
                )], 
                // Ruta del archivo WAR generado por Maven (ej. /target/Spring5.war)
                // Ajusta 'Spring5.war' al nombre exacto de tu artefacto
                contextPath: "${params.TOMCAT_APP_NAME}",
                war: 'target/**/*.war' 
            }
        }
    }
    
    // Acciones a realizar después de que el Pipeline ha terminado
    post {
        always {
            echo 'Pipeline finalizado. Verificando estado...'
        }
        success {
            echo '✅ Despliegue exitoso.'
        }
        failure {
            echo '❌ El Pipeline falló. Revisa los logs.'
        }
    }
}