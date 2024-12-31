pipeline {
    agent any

    tools {
        maven 'MVN'
    }

    environment {
        GIT_REPO_URL = 'https://github.com/1182147/ODSOFT_2425_1182147_1190409_AUTHNUSERS.git'
        GIT_BRANCH = 'main'

        TEST_SERVER_PORT = 'to-be-filled'
        PROD_SERVER_PORT = '2228'

        IMAGE_NAME = 'lmsauthnusers'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
            }
        }

        stage('Clean Install') {
            steps {
                script {
                    if(isUnix()) {
                        sh "mvn clean compile test-compile"
                    } else {
                        bat "mvn clean compile test-compile"
                    }
                }
            }
        }

        // For rollback purposes it is always valuable to keep the artifact of
        // the built binary.
        stage('Archive JAR Artifact') {
             steps {
                  archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
             }
        }

//         stage('Deploy Development Environment') {
//         }
//
//         stage('Deploy Testing Environment') {
//         }

        stage('Deploy Production Environment') {
            // This mimics the 'CREATE IF NOT EXISTS' which POSTGRES does not have.
            // POSTGRES SHIT = psql -U postgres -tc "SELECT 1 FROM pg_database WHERE datname = 'users'" | grep -q 1 && echo "Database 'users' already initialized" || psql -U postgres -c "CREATE DATABASE users"
            // POSTGRES SHIT = psql -U postgres -tc "SELECT 1 FROM pg_database WHERE datname = 'books'" | grep -q 1 && echo "Database 'books' already initialized" || psql -U postgres -c "CREATE DATABASE books"
        }

        stage('Deploy') {
            steps {
                script {
                    if(isUnix()) {
                        sh "JENKINS_NODE_COOKIE=dontKillMe java -jar ./target/psoft-g1-0.0.1-SNAPSHOT.jar --server.port=2228 > output.log 2>&1 &"
                    } else {
                        bat "set JENKINS_NODE_COOKIE=dontKillMe && start java -jar .\\target\\psoft-g1-0.0.1-SNAPSHOT.jar --server.port=2228 > output.log 2>&1"
                    }
                }
            }
        }
    }
}
