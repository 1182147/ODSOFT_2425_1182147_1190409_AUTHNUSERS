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

        IMAGE_NAME = 'lmsusers'
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
                    sh "mvn clean install"
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t lmsusers ."
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

        stage('Deploy Production Infrastructure') {
            script {
                sh """
                    if ! docker ps --format '{{.Names}}' | grep -q rabbitmq_in_lms_network; then
                        docker compose -f docker-compose-rabbitmq+postgres.yml up -d
                    else
                        echo "RabbitMQ container already running."
                    fi

                    if ! docker ps --format '{{.Names}}' | grep -q postgres_in_lms_network; then
                        docker compose -f docker-compose-rabbitmq+postgres.yml up -d
                    else
                        echo "Postgres container already running."
                    fi

                    for i in $(seq 1 5)
                    do
                        echo "Attempt $i to Health-Check Postgres Container"
                        if docker inspect --format='{{json .State.Health.Status}}' postgres_in_lms_network | grep healthy; then
                            docker exec -it postgres_in_lms_network /bin/sh -c 'psql -U postgres -tc "SELECT 1 FROM pg_database WHERE datname = '\''users'\''" | grep -q 1 && echo "Database '\''users'\'' already initialized" || psql -U postgres -c "CREATE DATABASE users;"'
                            docker exec -it postgres_in_lms_network /bin/sh -c 'psql -U postgres -tc "SELECT 1 FROM pg_database WHERE datname = '\''books'\''" | grep -q 1 && echo "Database '\''books'\'' already initialized" || psql -U postgres -c "CREATE DATABASE books;"'
                            exit 0
                        fi
                        sleep 10
                    done
                    echo "Postgres container failed to pass HealthChecks."
                    exit 1
                """
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh "docker compose up -d --force-recreate"
                }
            }
        }

        post {
            success {
                echo 'Pipeline completed successfully!'
            }
            failure {
                echo 'Pipeline failed!'
            }
        }
    }
}
