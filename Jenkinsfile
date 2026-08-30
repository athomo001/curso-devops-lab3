// Pipeline "declarativo": define etapas (stages) en orden, cada una aparece
// como un bloque en el Stage View de Jenkins
pipeline {
  // Corre en cualquier agente/nodo disponible de Jenkins
  agent any
  tools {
    nodejs 'Node20'
  }

  // Variables de entorno visibles en todos los stages, como constantes del pipeline
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

    // 2) Instala las dependencias exactas fijadas en package-lock.json
    stage('Instalación de dependencias') {
      steps { sh 'npm ci' }
    }

    // 3) Corre los tests unitarios y genera coverage/lcov.info
    stage('Ejecución de pruebas') {
      steps { sh 'npm run test:cov' }
    }

    // 4) Compila TypeScript -> JavaScript, para detectar errores de build
    // antes de gastar tiempo armando la imagen Docker
    stage('Build de la aplicación') {
      steps { sh 'npm run build' }
    }

    // 5) Lee la versión de package.json para usarla como uno de los 3 tags
    stage('Definir versión semántica') {
      steps {
        script {
          // sh(...) con returnStdout:true captura la salida del comando en
          // una variable de Groovy en vez de solo mostrarla en el log
          env.APP_VERSION = sh(
            script: "node -p \"require('./package.json').version\"",
            returnStdout: true
          ).trim()
        }
      }
    }

    // 6) Construye la imagen multistage; BUILD_NUMBER es una variable que
    // Jenkins ya trae incorporada (se autoincrementa en cada ejecución)
    stage('Construcción imagen Docker (multistage)') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
      }
    }
  }
}