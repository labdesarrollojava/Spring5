pipeline {
    agent any

    parameters {
        // Establecer un valor por defecto sin '/' para mayor flexibilidad
        string(name: 'TOMCAT_APP_NAME', defaultValue: 'Spring5', description: 'Nombre de la aplicación en Tomcat (context path)')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Obteniendo código del repositorio...'
                // Si la configuración del Job ya tiene Git, 'checkout scm' lo usa.
                checkout scm 
            }
        }

        stage('Build WAR') {
            steps {
                echo 'Compilando y empaquetando el proyecto con Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Iniciando el despliegue en Tomcat...'

                // 🚨 ¡LA CORRECCIÓN ESTÁ AQUÍ! 
                // El paso 'deploy' debe agrupar todos sus argumentos.
                deploy(
                    // Usa 'target/*.war' para asegurar que se encuentra el artefacto compilado
                    war: 'target/*.war', 
                    
                    // Define el Context Path. El plugin generalmente requiere el '/' inicial.
                    contextPath: "/${params.TOMCAT_APP_NAME}", 
                    
                    adapters: [
                        tomcat8( 
                            credentialsId: 'tomcat-auth',
                            url: 'http://tomcat:8080'  
                        )
                    ]
                )
            }
        }
    }
    
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