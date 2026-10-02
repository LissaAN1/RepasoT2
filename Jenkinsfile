// =============================================================================
// Taller Evaluativo 2 - Fase 4: Pipeline Declarativo en Jenkins
//
// Complete los TODOs de cada etapa. Recuerde:
//   - Prohibido el tag ":latest" al publicar en Nexus (empaquetamiento
//     inmutable, Fase 2).
//   - Las credenciales (usuario/clave de Nexus, token de GitHub) se
//     inyectan mediante los IDs configurados en Jenkins > Credentials,
//     NUNCA en texto plano dentro de este archivo.
// =============================================================================

pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        githubPush()
    }

    environment {
        // TODO
        IMAGE_TAG = "TODO-DEFINA-EN-UN-STAGE-SCRIPT"

        // TODO
        NEXUS_REGISTRY   = "localhost:9080"
        NEXUS_MAVEN_REPO = "http://nexus:8081/repository/maven-releases/"

        // TODO
        NEXUS_CREDENTIALS_ID = "nexus-credentials"
    }

    stages {

        stage('Checkout & Test') {
            steps {
                checkout scm

                script {
                    // TODO:
                    def shortCommit = sh(script: 'git rev-parse --short=7 HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${shortCommit}"
                    echo "IMAGE_TAG definido: ${env.IMAGE_TAG}"
                }

                dir('backend') {
                    // TODO
                    sh 'mvn -B test'
                }
            }
        }

        stage('Package & Tag Inmutable') {
            steps {
                dir('backend') {
                    // TODO
                    sh "mvn -B package -DskipTests -Drevision=${env.IMAGE_TAG}"
                    sh "docker build -t ${env.NEXUS_REGISTRY}/studytrack-backend:${env.IMAGE_TAG} ."
                }
                dir('frontend') {
                    // TODO:
                    sh "docker build --build-arg VITE_API_URL=http://localhost:8080 -t ${env.NEXUS_REGISTRY}/studytrack-frontend:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                // TODO
                withCredentials([usernamePassword(
                    credentialsId: env.NEXUS_CREDENTIALS_ID,
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS')]) {

                writeFile file: 'ci-settings.xml', text: '''<settings>
    <servers>
      <server>
        <id>nexus</id>
        <username>${env.NEXUS_USER}</username>
        <password>${env.NEXUS_PASS}</password>
      </server>
    </servers>
  </settings>'''

                    // Imágenes Docker -> docker-hosted
                    sh 'echo "$NEXUS_PASS" | docker login "$NEXUS_REGISTRY" -u "$NEXUS_USER" --password-stdin'
                    sh "docker push ${env.NEXUS_REGISTRY}/studytrack-backend:${env.IMAGE_TAG}"
                    sh "docker push ${env.NEXUS_REGISTRY}/studytrack-frontend:${env.IMAGE_TAG}"

                    // JAR -> maven-releases
                    dir('backend') {
                        sh "mvn -B deploy -DskipTests -s ../ci-settings.xml -Drevision=${env.IMAGE_TAG} -Dnexus.maven.url=${env.NEXUS_MAVEN_REPO}"
                    }
                }
            }
        }

        stage('Deploy & Smoke Test') {
            steps {
                // TODO
                withCredentials([usernamePassword(
                        credentialsId: env.NEXUS_CREDENTIALS_ID,
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS')]) {
                    sh 'echo "$NEXUS_PASS" | docker login "$NEXUS_REGISTRY" -u "$NEXUS_USER" --password-stdin'
                    sh 'docker compose -f deploy/docker-compose.yml pull'
                    sh 'docker compose -f deploy/docker-compose.yml up -d'
                }
                sh '''
                    for i in $(seq 1 15); do
                        code=$(curl -s -o /dev/null -w "%{http_code}" http://studytrack-backend:8080/api/tasks || true)
                        if [ "$code" = "200" ]; then
                            echo "Smoke test OK (HTTP 200) en el intento $i"
                            exit 0
                        fi
                        echo "Intento $i/15: HTTP $code - reintentando en 5s..."
                        sleep 5
                    done
                    echo "Smoke test FALLÓ: /api/tasks no respondió 200"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline finalizado en verde. Artefactos publicados con tag: ${env.IMAGE_TAG}"
        }
        failure {
            echo "El pipeline falló. Revise los logs de la etapa correspondiente antes de reintentar."
        }
    }
}
