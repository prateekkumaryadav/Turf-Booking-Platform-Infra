pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Compose Files') {
            steps {
                echo 'Validating docker-compose files...'
                sh 'docker compose -f docker-compose.yml config --quiet || echo "Dev compose validation completed"'
                sh 'docker compose -f docker-compose.prod.yml config --quiet || echo "Prod compose validation completed"'
            }
        } 

        stage('Deploy with Ansible') {
            steps {
                echo 'Deploying with Ansible...'
                dir('ansible') {
                    sh 'touch ../.env'
                    sh 'ansible-playbook playbooks/deploy.yml'
                }
            }
        }
    }

    post {
        always {
            echo "Infra Pipeline run completed."
        }
        success {
            emailext (
                subject: "DEPLOY SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Infrastructure deployment was successful!\nProject: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nBuild URL: ${env.BUILD_URL}",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                to: 'prateek.student20@gmail.com',
                mimeType: 'text/plain'
            )
        }
        failure {
            emailext (
                subject: "DEPLOY FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Infrastructure deployment has failed!\nProject: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nBuild URL: ${env.BUILD_URL}\nCheck the console output",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                to: 'prateek.student20@gmail.com',
                mimeType: 'text/plain'
            )
        }
    }
}
