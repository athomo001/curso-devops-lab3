pipeline {
  agent any

  environment {
    //DOCKERHUB_USER = '<DOCKERHUB_USER>'   // tu usuario de Docker Hub
    //GHCR_USER      = '<GITHUB_USER>'      // tu usuario/organización de GitHub (para ghcr.io)
    IMAGE_NAME     = 'curso-devops-lab3'  // nombre base de la imagen
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
        stage('SonarQube - análisis de codigo') {
          steps {
            withSonarQubeEnv('sonarqube') {
              sh 'sonar-scanner'
            }
          }
        }

        stage('Quality Gate') {
          steps {
            timeout(time: 5, unit: 'MINUTES') {
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

    // TODO (Paso 4.3): Publicar en Docker Hub,
    //   una vez que crees la credencial 'dockerhub-creds' (Paso 3)
    // TODO (Paso 4.4): Publicar en GitHub Packages (GHCR),
    //   una vez que crees la credencial 'github-creds' (Paso 3)
    // TODO (Paso 4.5): Actualizar imagen en Kubernetes,
    //   una vez que tengas el cluster arriba y kubernetes.yaml aplicado (Paso 5)
  }
}
