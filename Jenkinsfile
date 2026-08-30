
def tagAndpush(String localImage, String repo, String registry, String credential) {
  docker.withRegistry(registry, credential) {
    sh "docker tag ${localImage}:${env.BUILD_NUMBER} ${repo}:latest"
    sh "docker tag ${localImage}:${env.BUILD_NUMBER} ${repo}:${env.BUILD_NUMBER}"
    sh "docker tag ${localImage}:${env.BUILD_NUMBER} ${repo}:${env.APP_VERSION}"
    sh "docker push ${repo}:latest"
    sh "docker push ${repo}:${env.BUILD_NUMBER}"
    sh "docker push ${repo}:${env.APP_VERSION}"
  }
}

pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'athanespinoza'   // usuario de Docker Hub
    GHCR_USER      = 'athomo001'       // usuario/organización de GitHub (para ghcr.io)
    IMAGE_NAME     = 'curso-devops-lab3'  // nombre base de la imagen
    DH_REPO    = "athanespinoza/curso-devops-lab3"
    GHCR_REPO  = "ghcr.io/athomo001/curso-devops-lab3"
    //K8S_NAMESPACE  = '<TUNAMESPACE>'      // namespace de Kubernetes a actualizar
    //K8S_DEPLOYMENT = 'curso-devops-lab3'  // nombre del Deployment en kubernetes.yaml
    //K8S_CONTAINER  = 'curso-devops-lab3'  // nombre del contenedor dentro del Deployment
  }

  stages {
    // 1) Descarga el código del repositorio configurado en el Job de Jenkins
    stage('Checkout') {
      steps { checkout scm }
    }


    stage('CI - Integración continua') {
      agent {
        docker {
            image 'node:20-alpine';
            reuseNode true
            }
        }
      stages {
        // Instala las dependencias exactas fijadas en package-lock.json
        stage('Instalación de dependencias') {
          steps { sh 'npm ci' }
        }

        // Corre los tests unitarios y genera coverage/lcov.info
        stage('Ejecución de pruebas') {
          steps { sh 'npm run test:cov' }
        }

        stage('Build de la aplicación') {
          steps { sh 'npm run build' }
        }

        stage('Definir versión semántica') {
          steps {
            script {

              env.APP_VERSION = sh(
                script: "node -p \"require('./package.json').version\"",
                returnStdout: true
              ).trim()
            }
          }
        }
      }
    }

    stage('Quality Assurance') {
      agent {
        docker {
          image 'sonarsource/sonar-scanner-cli'
          args '--network devops-infra_default'
          reuseNode true
        }
      }
      stages {
        stage('SonarQube - análisis de código') {
          steps {
            withSonarQubeEnv('sonarqube') {
              sh 'sonar-scanner'
            }
          }
        }

        stage('Quality Gate') {
          steps {
            timeout(time: 1, unit: 'MINUTES') {
              waitForQualityGate abortPipeline: true
            }
          }
        }
      }
    }

    stage('Construcción imagen Docker (multistage)') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
      }
    }

    stage('Publicar en Docker Hub') {       
      steps {
        sh "docker build -t ${env.IMAGE_NAME} ."
         script {
            if (!env.APP_SEMANTIC_VERSION?.trim()) {
                error("APP_SEMANTIC_VERSION no definida en el stage anterior")
                }
          tagAndpush(env.IMAGE_NAME, env.DH_REPO, 'https://index.docker.io/v1/', 'curso-devops-dh')
        }
      }
    }

    // Mismo patrón que el stage anterior, pero apuntando al registry de
    // GitHub (ghcr.io) en vez de Docker Hub
    stage('Publicar en GitHub Packages (GHCR)') {
      steps {
        sh "docker build -t ${env.IMAGE_NAME} ."
         script {
            if (!env.APP_SEMANTIC_VERSION?.trim()) {
                error("APP_SEMANTIC_VERSION no definida en el stage anterior")
                }
          tagAndpush(env.IMAGE_NAME, env.GHCR_REPO, 'https://ghcr.io', 'curso-devops-gh')
        }
      }
    }

    // TODO (Paso 4.5): Actualizar imagen en Kubernetes,
    //   una vez que tengas el cluster arriba y kubernetes.yaml aplicado (Paso 5)
  }
}
