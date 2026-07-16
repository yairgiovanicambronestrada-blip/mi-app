pipeline {
    agent any

    environment {
        GITHUB_USER = 'yairgiovanicambronestrada-blip'   // ← CAMBIA A TU USUARIO
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'yairgiovanicambronestrada-blip/mi-app'  // ← CAMBIA A TU USUARIO
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
                branch 'main'   // ← Si tu rama es 'master', cámbialo aquí
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
        cleanup { sh 'docker image prune -f' }
    }
}
