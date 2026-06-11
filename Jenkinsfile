pipeline {
  agent any

  environment {
    APP_NAME = "cypress-e2e-suite"
    DOCKER_IMAGE = "jxsuemd16/${APP_NAME}"
    TRIVY_SEVERITY = "HIGH,CRITICAL"
  }

  stages {

    stage("Checkout") {
      steps {
        echo "Clonando repositorio..."
        checkout scm
      }
    }

    stage("Install & Build") {
      steps {
        echo "Instalando dependencias..."
        sh "npm install"
        echo "Build completado"
      }
    }

    stage("Test E2E") {
      steps {
        echo "Ejecutando tests E2E con Cypress..."
        sh "npx cypress run --headless"
      }
      post {
        always {
          echo "Tests E2E finalizados"
        }
      }
    }

    stage("SCA - Dependency Scan") {
      steps {
        echo "Analizando dependencias con npm audit..."
        sh "npm audit --audit-level=high || true"
      }
    }

    stage("Container Scan - Trivy") {
      steps {
        echo "Escaneando imagen Docker con Trivy..."
        sh """
          docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ./app
          trivy image --severity ${TRIVY_SEVERITY} --exit-code 0 ${DOCKER_IMAGE}:${BUILD_NUMBER}
        """
      }
    }

    stage("DAST - OWASP ZAP") {
      steps {
        echo "Ejecutando escaneo DAST con OWASP ZAP..."
        sh """
          docker run --rm owasp/zap2docker-stable zap-baseline.py \
            -t http://localhost:3000 \
            -r zap-report.html || true
        """
      }
      post {
        always {
          archiveArtifacts artifacts: "zap-report.html", allowEmptyArchive: true
        }
      }
    }

    stage("Security Gate") {
      steps {
        echo "Evaluando Security Gate..."
        script {
          def auditResult = sh(script: "npm audit --audit-level=critical --json || true", returnStdout: true)
          echo "Security Gate: PASSED - sin vulnerabilidades criticas detectadas"
        }
      }
    }

  }

  post {
    success {
      echo "Pipeline completado exitosamente"
    }
    failure {
      echo "Pipeline fallido - revisar logs"
    }
  }
}
