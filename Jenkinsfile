pipeline {
    agent none

    stages {

        stage('Checkout') {
            agent any
            steps {
                // Nettoyer le workspace avant checkout
                sh 'rm -rf target || true'
                checkout scm
                sh 'git log --oneline -5'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.7-eclipse-temurin-11'
                    // Supprimer -u root → utiliser le meme user que le host
                    args '-v $HOME/.m2:/root/.m2 --user 1000:1000'
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
                    args '-v $HOME/.m2:/root/.m2 --user 1000:1000'
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
                    args '-v $HOME/.m2:/root/.m2 --user 1000:1000'
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
            // node obligatoire car agent none au niveau pipeline
            node('built-in') {
                echo "✅ Build #${BUILD_NUMBER} reussi !"
                mail to: 'elharitihassane@gmail.com',
                     subject: "✅ Build Passed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Build successful! Check: ${env.BUILD_URL}"
            }
        }
        failure {
            node('built-in') {
                mail to: 'elharitihassane@gmail.com',
                     subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: """
                       Build failed!
                       Job: ${env.JOB_NAME}
                       Build: ${env.BUILD_NUMBER}
                       URL: ${env.BUILD_URL}
                     """
            }
        }
        always {
            node('built-in') {
                cleanWs()    // ✅ maintenant cleanWs a un node context
            }
        }
    }
}