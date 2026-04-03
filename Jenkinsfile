pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Tests unitaires') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Build #${BUILD_NUMBER} réussi !"
            mail to: 'elharitihassane@gmail.com',
             subject: "✅ Build Passed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "Build was successful. Check: ${env.BUILD_URL}"

        }
        failure {                                    // ← this was missing!
            mail to: 'elharitihassane@gmail.com',
                 subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """
                   Build failed!
                   Job: ${env.JOB_NAME}
                   Build: ${env.BUILD_NUMBER}
                   URL: ${env.BUILD_URL}
                 """
        }
        always {
            cleanWs()
        }
    }
}
