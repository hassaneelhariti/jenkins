pipeline {
    agent any

    stages {
        stage('Checkout') {
            agent {
                docker {
                    image 'alpine/git'         
                    args '-v /tmp/workspace:/workspace'
                }
            }
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.7-openjdk-11'   
                    args '-v $HOME/.m2:/root/.m2'     
                }
            }
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Tests unitaires') {
            agent {
                docker {
                    image 'maven:3.8.7-openjdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
                success {
                    echo '✅ Tous les tests sont passes !'
                }
                failure {
                    echo '❌ Des tests ont echoue !'
                }
            }
        }

        stage('Package') {
            agent {
                docker {
                    image 'maven:3.8.7-openjdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
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
