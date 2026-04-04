pipeline {
    agent none

    stages {

        stage('Checkout') {
            agent any
            steps {
                checkout scm
                sh 'git log --oneline -5'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.7-eclipse-temurin-11'
                    args '-u root -v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean compile -B'
            }
        }

        stage('Tests Unitaires') {
            agent {
                docker {
                    image 'maven:3.8.7-eclipse-temurin-11'
                    args '-u root -v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            agent {
                docker {
                    image 'maven:3.8.7-eclipse-temurin-11'
                    args '-u root -v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn package -DskipTests -B'
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

    }

    post {
        success {
            mail to: 'elharitihassane@gmail.com',
                 subject: "✅ Build Passed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build successful! Check: ${env.BUILD_URL}"
        }
        failure {
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