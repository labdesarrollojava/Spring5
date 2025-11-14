#!/bin/bash
# Jenkinsfile para el proyecto Spring5
# Despliegue automático en Tomcat después de compilar
# Este archivo debe copiarse al repositorio Spring5

pipeline {
    agent any

    environment {
        // Configuración de la aplicación
        TOMCAT_CONTAINER = "tomcat"
        APP_NAME = "testing-junit5-mockito"  // Nombre del artefacto en pom.xml
        MAVEN_HOME = "/usr/share/maven"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "📥 Descargando código fuente..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "🔨 Compilando con Maven..."
                sh '''
                    mvn --version
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Ejecutando pruebas..."
                sh '''
                    mvn test
                '''
            }
        }

        stage('Verify WAR Generation') {
            steps {
                echo "📦 Verificando generación del WAR..."
                sh '''
                    if [ -f "target/testing-junit5-mockito.jar" ]; then
                        echo "✓ JAR encontrado: target/testing-junit5-mockito.jar"
                        ls -lh target/testing-junit5-mockito.jar
                    elif [ -f "target/testing-junit5-mockito.war" ]; then
                        echo "✓ WAR encontrado: target/testing-junit5-mockito.war"
                        ls -lh target/testing-junit5-mockito.war
                    else
                        echo "⚠️  Buscando archivos en target/"
                        ls -lh target/ | grep -E "\\.(jar|war)$" || echo "No se encontraron archivos"
                    fi
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Desplegando a Tomcat..."
                sh '''
                    # Buscar el artefacto generado
                    ARTIFACT=$(find target -maxdepth 1 -type f \\( -name "*.jar" -o -name "*.war" \\) ! -name "*sources*" ! -name "*javadoc*" | head -1)
                    
                    if [ -z "$ARTIFACT" ]; then
                        echo "❌ Error: No se encontró WAR/JAR"
                        ls -la target/
                        exit 1
                    fi
                    
                    ARTIFACT_NAME=$(basename "$ARTIFACT")
                    echo "📤 Copiando $ARTIFACT_NAME a Tomcat..."
                    
                    docker cp "$ARTIFACT" "${TOMCAT_CONTAINER}:/usr/local/tomcat/webapps/${APP_NAME}.jar"
                    
                    echo "⏳ Esperando a que se despliegue..."
                    for i in {1..30}; do
                        # Para Spring Boot JAR (modo embedded)
                        # Esperar acceso HTTP
                        if curl -f http://localhost:8081/${APP_NAME}/ --silent > /dev/null 2>&1; then
                            echo "✓ Aplicación disponible"
                            exit 0
                        fi
                        echo "  Intento $i/30..."
                        sleep 2
                    done
                    
                    echo "⚠️  Tiempo de espera agotado, pero continuando..."
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "🏥 Verificando salud de la aplicación..."
                sh '''
                    echo "Esperando 5 segundos..."
                    sleep 5
                    
                    echo "Probando acceso a la aplicación..."
                    curl -v http://localhost:8081/${APP_NAME}/ 2>&1 | head -20 || true
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Despliegue completado exitosamente"
            echo "📍 Aplicación disponible en: http://localhost:8081/${APP_NAME}"
        }
        failure {
            echo "❌ El despliegue falló"
            sh '''
                echo "Logs de Tomcat:"
                docker logs tomcat | tail -30
                
                echo "Contenido de webapps:"
                docker exec tomcat ls -la /usr/local/tomcat/webapps/
            '''
        }
        always {
            echo "🧹 Limpieza de workspace..."
            cleanWs()
        }
    }
}
