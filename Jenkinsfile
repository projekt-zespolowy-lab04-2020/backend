pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        withGradle() {
          sh 'chmod +x gradlew'
        }

      }
    }

    stage('Test') {
      steps {
        withGradle() {
          sh './gradlew clean ktlintCheck test -x :smoke:test --info'
        }

      }
    }

    stage('Build jar') {
      steps {
        withGradle() {
          sh './gradlew clean shadowJar'
        }

      }
    }

    stage('Docker prepare PRE') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        sh 'docker stop backendruntest || true'
        sh 'docker rm backendruntest  || true'
        sh 'docker image rm backendtest || true'
      }
    }

    stage('Docker build PRE') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        dir(path: '/var/lib/jenkins/workspace/backend_master/target/') {
          sh 'docker build . -t backendtest'
        }

      }
    }

    stage('Docker run PRE') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        sh 'docker run -d -p 8091:8090 -e DATABASE_HOST=172.18.0.4:5432 -e DATABASE_NAME=tourtool_pre -e DATABASE_USER -e DATABASE_PASSWORD -e APP_SECRET -e STAGE=STA --name backendruntest --restart always --net netapp -it backendtest'
      }
    }
    stage('Smoke Test') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        withGradle() {
          sh './gradlew smoke:test'
        }

      }
    }
    stage('Ask Promote') {
      when {
        anyOf {
          branch 'master'
        }
      }
      steps {
        script {
          def userInput = input(
            id: 'userInput', message: 'Promote this build?', parameters: [
                [$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you are sure to proceed']
            ]
          )
          if (!userInput) {
              currentBuild.result = 'ABORTED'
              error "Build wasn't promoted"
          }
        }
      }
    }
    stage('Docker prepare PRO') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        sh 'docker stop backendrun || true'
        sh 'docker rm backendrun  || true'
        sh 'docker image rm backend || true'
      }
    }

    stage('Docker build PRO') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        dir(path: '/var/lib/jenkins/workspace/backend_master/target/') {
          sh 'docker build . -t backend'
        }

      }
    }

    stage('Docker run PRO') {
      when {
        anyOf {
          branch 'master'
        }

      }
      steps {
        sh 'docker run -d -p 8090:8090 -e DATABASE_HOST=172.18.0.4:5432 -e DATABASE_NAME=tourtool -e DATABASE_USER -e DATABASE_PASSWORD -e APP_SECRET -e STAGE=PRO --name backendrun --restart always --net netapp -it backend'
      }
    }

  }
  environment {
    DATABASE_USER = 'backend'
    DATABASE_PASSWORD = credentials('database-password')
    APP_SECRET = credentials('app-secret')
    USER_EMAIL = credentials('user-email')
    USER_PASSWORD = credentials('user-password')
  }
}
