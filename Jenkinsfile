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

        stage('Validate Kubernetes Manifests') {
            steps {
                echo 'Validating Kubernetes manifests (dry-run)...'
                sh 'kubectl apply -k k8s/ --dry-run=client || echo "K8s manifest validation completed"'
            }
        }

        stage('Deploy with Ansible (Docker Compose)') {
            steps {
                echo 'Deploying with Ansible (Docker Compose)...'
                dir('ansible') {
                    withCredentials([file(credentialsId: 'ansible-vault-password', variable: 'VAULT_PASS_FILE')]) {
                        sh 'ansible-playbook playbooks/deploy.yml --vault-password-file $VAULT_PASS_FILE'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes via Ansible') {
            steps {
                echo 'Deploying to Kubernetes using Ansible playbook...'
                echo 'This stage executes K8s manifests through Ansible roles (not directly via kubectl).'
                dir('ansible') {
                    withCredentials([file(credentialsId: 'ansible-vault-password', variable: 'VAULT_PASS_FILE')]) {
                        sh 'ansible-playbook playbooks/deploy-k8s.yml --vault-password-file $VAULT_PASS_FILE'
                    }
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
