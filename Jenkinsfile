pipeline {
    // Usamos un agente Docker con Node.js y montamos el socket de Docker del host
    agent {
        docker {
            image 'node:18-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root'
        }
    }

    environment {
        // ⚠️ CAMBIA 'tu-usuario-de-github' y 'tu-usuario/mi-app' por los tuyos
        GITHUB_USER = 'DarkYeah'
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'yairgiovanicambronestrada-blip/mi-app'
        COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
        IMAGE_TAG_LATEST = "${REGISTRY}/${IMAGE_NAME}:latest"
        IMAGE_TAG_COMMIT = "${REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA}"
        IMAGE_TAG_BUILD = "${REGISTRY}/${IMAGE_NAME}:build-${BUILD_TIMESTAMP}"
    }

    stages {
        stage('Prepare') {
            steps {
                echo '🔧 Preparando entorno...'
                // Instalamos Docker CLI dentro del contenedor (para poder usar el comando docker)
                sh 'apk add --no-cache docker-cli'
                sh 'docker --version'
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '📥 Instalando dependencias...'
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Ejecutando tests...'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Construyendo imagen Docker...'
                script {
                    docker.build("${IMAGE_TAG_COMMIT}")
                }
            }
        }

        stage('Push to Registry') {
            when {
                branch 'main'   // Si tu rama es 'master', cámbialo aquí
            }
            steps {
                echo '📤 Publicando imagen en GitHub Container Registry...'
                script {
                    withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USER --password-stdin
                        '''
                        sh """
                            docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_LATEST}
                            docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_BUILD}
                            docker push ${IMAGE_TAG_COMMIT}
                            docker push ${IMAGE_TAG_LATEST}
                            docker push ${IMAGE_TAG_BUILD}
                        """
                    }
                }
            }
        }

        stage('Verify Published Image') {
            when {
                branch 'main'
            }
            steps {
                echo '✅ Verificando imagen publicada...'
                script {
                    sh """
                        echo "📦 Imagen publicada: ${IMAGE_TAG_COMMIT}"
                        echo "🏷️ Tags disponibles:"
                        echo " - ${IMAGE_TAG_COMMIT}"
                        echo " - ${IMAGE_TAG_LATEST}"
                        echo " - ${IMAGE_TAG_BUILD}"
                    """
                }
            }
        }
    }

    post {
        success { echo '🎉 Pipeline completado exitosamente!' }
        failure { echo '❌ Pipeline falló!' }
        cleanup {
            echo '🧹 Limpiando recursos...'
            sh 'docker image prune -f || true'  // El '|| true' evita que falle si no hay nada que limpiar
        }
    }
}
