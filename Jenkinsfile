pipeline {
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    } 
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    agent any
    stages {
        stage('checkout') {
            steps {
                script {
                    deleteDir()
                    retry(3) {
                        checkout([$class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[
                                url: 'https://github.com/Naval4206/Terraform-Jenkins.git',
                                credentialsId: 'your-credential-id'
                            ]],
                            extensions: [[$class: 'CloneOption', timeout: 30, noTags: false]]
                        ])
                    }
                }
            }
        }

        stage('Verify Files') {
            steps {
                sh 'ls -la'  // Verify if terraform directory exists
            }
        }

        stage('Plan') {
            steps {
                sh '''
                pwd
                ls -la  # Check if terraform directory is present
                if [ -d "terraform" ]; then
                    cd terraform
                    terraform init
                    terraform plan -out tfplan
                    terraform show -no-color tfplan > tfplan.txt
                else
                    echo "Error: terraform/ directory not found!"
                    exit 1
                fi
                '''
            }
        }

        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
           }
           steps {
               script {
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            steps {
                sh '''
                if [ -d "terraform" ]; then
                    cd terraform
                    terraform apply -input=false tfplan
                else
                    echo "Error: terraform/ directory not found!"
                    exit 1
                fi
                '''
            }
        }
    }
}
