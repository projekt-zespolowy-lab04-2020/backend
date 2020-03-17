pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        echo 'Preparing...'
        withGradle() {
          sh 'chmod +x gradlew'
        }

        echo 'Preparing ended'
      }
    }

    stage('Test') {
      steps {
        echo 'Start test'
        withGradle() {
          sh './gradlew ktlintCheck test'
        }

        echo 'End test'
      }
    }

    stage('Build jar') {
      steps {
        echo 'Starting build'
        withGradle() {
          sh './gradlew clean build'
        }

        echo 'Build ended'
      }
    }

    stage('Docker prepare') {
      steps {
        sh 'docker container stop backendrun'
        sh 'docker container rm backendrun'
        sh 'docker image rm backend'
      }
    }

    stage('Migrate database') {
      steps {
        withGradle() {
          sh './gradlew flywayMigrate'
        }
      }
    }

    stage('Docker build') {
      steps {
        dir(path: '/var/lib/jenkins/workspace/backend_master/target/') {
          sh 'docker build . -t backend'
        }

      }
    }

    stage('Docker run') {
      steps {
        sh 'docker run -d -p 8090:8090 --name backendrun -it backend'
      }
    }

  }
}
