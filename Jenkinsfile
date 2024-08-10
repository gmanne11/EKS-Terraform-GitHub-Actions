properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )])
])
pipeline {
    agent any
    stages {
        stage('Preparing') {
            steps {
                sh 'echo Preparing'
            }
        }
         stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'git-creds', url: 'https://github.com/gmanne11/EKS-Terraform-GitHub-Actions.git'
            }
        }
        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh 'terraform -chdir=eks/ init'
                }
            }
        }
        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh 'terraform -chdir=eks/ validate'
                }
            }
        }
        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {    
                        if (params.Terraform_Action == 'plan') {
                            sh "terraform -chdir=eks/ plan -var-file=${params.Environment}.tfvars"
                        } else if (params.Terraform_Action == 'apply') {
                            sh "terraform -chdir=eks/ apply -var-file=${params.Environment}.tfvars -auto-approve"
                        } else if (params.Terraform_Action == 'destroy') {
                            sh "terraform -chdir=eks/ destroy -var-file=${params.Environment}.tfvars -auto-approve"
                        } else {
                            error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                // Send email on success
                emailext (
                    subject: "Jenkins Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "The pipeline has completed successfully.\n\nCheck console output at ${env.BUILD_URL}",
                    recipientProviders: [[$class: 'CulpritRecipientProvider'], [$class: 'RequesterRecipientProvider']]
                )
            }
        }
        failure {
            script {
                // Send email on failure
                emailext (
                    subject: "Jenkins Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "The pipeline has failed.\n\nCheck console output at ${env.BUILD_URL}",
                    recipientProviders: [[$class: 'CulpritRecipientProvider'], [$class: 'RequesterRecipientProvider']]
                )
            }
        }
    }
}
