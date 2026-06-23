pipeline {
  agent any

  // Use Jenkins tool installations by name (Manage Jenkins -> Tools)
  tools {
    jdk 'jdk17'        // Change to your configured JDK name
    maven 'maven3'     // Change to your configured Maven name
  }

  options {
    // Keep logs concise and build discards; add timestamps to logs
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '10'))
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    // Set JAVA_HOME & PATH automatically from tools; add any env vars you need
    // Example: MAVEN_OPTS for memory tuning or proxies
    MAVEN_OPTS = '-Dmaven.test.failure.ignore=false'
  }

  triggers {
    // Optional: Poll SCM every 5 minutes or use GitHub/GitLab webhooks instead
    // pollSCM('H/5 * * * *') 
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm   // Uses Jenkins job's SCM configuration
        // Or: git url: 'https://your.git/repo.git', branch: 'main', credentialsId: 'your-creds'
      }
    }

    stage('Set Version (optional)') {
      when { expression { fileExists('pom.xml') } }
      steps {
        sh 'mvn -q --batch-mode help:evaluate -Dexpression=project.version -DforceStdout'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -U clean compile'   // -B batch mode; -U force update snapshots
      }
    }

    stage('Unit Tests') {
      steps {
        sh 'mvn -B test'
      }
      post {
        always {
          // Publish JUnit results produced by surefire
          junit testResults: '*/target/surefire-reports/*.xml, target/surefire-reports/*.xml', allowEmptyResults: true
        }
      }
    }

    stage('Package') {
      steps {
        sh 'mvn -B package'
      }
      post {
        success {
          // Archive JAR/WAR artifacts
          archiveArtifacts artifacts: 'target/*.jar, target/*.war', fingerprint: true
        }
      }
    }

    // Optional quality gate (requires SonarQube plugin & server config)
    // stage('SonarQube Analysis') {
    //   environment {
    //     // Name of your Sonar server in Jenkins global config
    //     // SONARQUBE_ENV is injected by withSonarQubeEnv step
    //   }
    //   steps {
    //     withSonarQubeEnv('Your-SonarQube-Server') {
    //       sh 'mvn -B verify sonar:sonar'
    //     }
    //   }
    // }

    // Optional: Build Docker image if your project has a Dockerfile
    // stage('Docker Build') {
    //   steps {
    //     sh 'docker build -t yourrepo/yourapp:${BUILD_NUMBER} .'
    //   }
    // }

    // Optional: Deploy (copy artifact, call kubectl/helm, or use SSH)
    // stage('Deploy to Dev') {
    //   when { branch 'main' }
    //   steps {
    //     // Example: copy artifact to server or run helm upgrade
    //     // sh 'scp target/yourapp.jar user@server:/opt/apps/'
    //     // sh 'kubectl apply -f k8s/deployment.yaml'
    //   }
    // }

  } // stages

  post {
    success {
      echo "✅ Build #${env.BUILD_NUMBER} succeeded."
    }
    failure {
      echo "❌ Build #${env.BUILD_NUMBER} failed."
    }
    always {
      // Useful diagnostics
      echo "Workspace: ${env.WORKSPACE}"
      sh 'ls -la'
    }
  }
}
