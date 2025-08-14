// Jenkinsfile (Declarative Pipeline)
// Adjust tool names ('MAVEN3', 'JDK17', 'sonarscanner') and Sonar server ('sonarserver') to match your Jenkins.

pipeline {
  agent any

  options {
    skipDefaultCheckout(true)
    timestamps()
  }

  tools {
    maven 'MAVEN3'   // Manage Jenkins -> Global Tool Configuration
    jdk   'JDK17'    // Manage Jenkins -> Global Tool Configuration
  }

  environment {
    SONARSERVER  = 'sonarserver'   // Manage Jenkins -> System (SonarQube servers)
    SCANNER_TOOL = 'sonarscanner'  // Name of SonarScanner tool in Global Tool Configuration
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        // Runs compile, tests, and generates JaCoCo XML at target/site/jacoco/jacoco.xml (if configured in pom.xml)
        sh 'mvn -s settings.xml -B clean verify'
      }
      post {
        success {
          echo 'Now Archiving.'
          archiveArtifacts artifacts: 'target/*.war', fingerprint: true, allowEmptyArchive: false
        }
      }
    }

    stage('Checkstyle Analysis') {
      steps {
        sh 'mvn -s settings.xml -B checkstyle:checkstyle'
      }
    }

    stage('Sonar Analysis') {
      steps {
        withSonarQubeEnv("${SONARSERVER}") {
          script { env.SCANNER_HOME = tool "${SCANNER_TOOL}" }
          sh """
            "\${SCANNER_HOME}/bin/sonar-scanner" \
              -Dsonar.projectKey=vprofile \
              -Dsonar.projectName=vprofile \
              -Dsonar.projectVersion=1.0 \
              -Dsonar.sources=src \
              -Dsonar.java.binaries=target/classes,target/test-classes \
              -Dsonar.junit.reportPaths=target/surefire-reports \
              -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
          """
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

  }
}
