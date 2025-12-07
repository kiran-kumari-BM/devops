pipeline {
  agent any

  // Optional: declare tools only if you rely on Jenkins-provided Maven/JDK.
  // We prefer the wrapper (./mvnw) so tools block is usually unnecessary.
  // tools {
  //   jdk 'jdk11'        // name configured in Jenkins Global Tool Config
  //   maven 'Maven 3.8'  // name configured in Jenkins Global Tool Config
  // }

  environment {
    // Use local .m2 inside workspace to avoid polluting global Jenkins node
    MAVEN_OPTS = "-Dmaven.repo.local=${env.WORKSPACE}/.m2/repository"
    // Optional: make sure non-interactive
    MAVEN_CLI_OPTS = "-B -DskipTests"
  }

  stages {
    stage('Checkout') {
      steps {
        // Checkout source as defined by Multibranch/SCM plugin
        checkout scm
        // Print a little info to help debugging
        echo "Checked out ${env.GIT_COMMIT} from ${env.GIT_URL ?: env.BRANCH_NAME}"
      }
    }

    stage('Verify Wrapper & Run Tests') {
      steps {
        // Ensure mvnw is executable (important on Unix agents)
        sh 'if [ -f mvnw ]; then chmod +x mvnw; fi'
        // Run a quick verify (runs tests by default; modify as needed)
        // Using wrapper if present; fallback to system mvn if not.
        sh '''
          if [ -x ./mvnw ]; then
            ./mvnw ${MAVEN_CLI_OPTS} verify
          else
            mvn ${MAVEN_CLI_OPTS} verify
          fi
        '''
      }
    }

    stage('Package') {
      steps {
        sh '''
          if [ -x ./mvnw ]; then
            ./mvnw -DskipTests clean package
          else
            mvn -DskipTests clean package
          fi
        '''
      }
    }

    stage('Archive artifact') {
      steps {
        // Archive produced jar(s) so they appear in Jenkins UI under build artifacts
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('List workspace') {
      steps {
        // Useful for debugging: show target/ and workspace contents
        sh 'ls -la .'
        sh 'ls -la target || true'
      }
    }
  }

  post {
    success {
      echo "Build successful — artifacts archived for ${env.BUILD_URL}"
    }
    failure {
      echo "Build failed — see console output: ${env.BUILD_URL}"
      // Optionally capture logs or run diagnostics here
    }
  }
}