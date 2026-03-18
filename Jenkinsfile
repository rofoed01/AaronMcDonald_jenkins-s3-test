pipeline {
    agent any
   
    environment {
        AWS_REGION = 'us-west-2' 
    }
    stages {
        
        // AWS Credentials documentation = https://plugins.jenkins.io/aws-credentials/
        stage('Set AWS Credentials') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsTest' // this needs to be changed before runtime to match the Jenkins credential
                ]]) {
                    sh '''
                    echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
                    aws sts get-caller-identity
                    '''
                }
            }
        }
        stage('Checkout Code') {
            steps {
                checkout scm 
            }
        }

        // stage('Testing') {
        //     // withEnv(["JFROG_BINARY_PATH=${tool 'jfrog-cli'}"]) {
        //     // // The 'jf' tool is available in this scope.
        //     // }
        //     steps {
        //         withCredentials([string(credentialsId: 'jfrog-creds', variable: 'JFROG_TOKEN')]) {
        //             // Show the installed version of JFrog CLI
        //             jf '-v'
                    
        //             // Show the configured JFrog Platform instances
        //             jf 'c show'
                    
        //             // Ping Artifactory
        //             jf 'rt ping'
                    
        //             // Create a file and upload it to the repository
        //             sh 'touch test-file'
        //             // Fixed upload command syntax
        //             sh 'jf rt upload test-file tf-terraform/ --url=https://trial7zoppg.jfrog.io/artifactory/ --user=mcdonald.dm.aaron@gmail.com --password=$JFROG_TOKEN'
                    
        //             // Publish the build-info to Artifactory
        //             jf 'rt bp'
                    
        //             // Fixed download command syntax
        //             sh 'jf rt download tf-terraform/test-file --url=https://trial7zoppg.jfrog.io/artifactory/ --user=mcdonald.dm.aaron@gmail.com --password=$JFROG_TOKEN'
        //         }
        //     } 
        // }
    
        stage('Initialize Terraform') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsTest'
                ]]) {
                    sh '''

                    terraform init
                    '''
                }
            }
        }

        stage('Validate Terraform') {
            steps {
                // terraform validate does NOT need credentials
                {
                    sh '''

                    terraform validate
                    '''
                }
            }
        }

        stage('Format Terraform') {
            steps {
                // terraform format does NOT need credentials
                {
                    sh '''

                    terraform fmt
                    '''
                }
            }
        }

        stage('Plan Terraform') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsTest'
                ]]) {
                    sh '''

                    terraform plan -out=tfplan
                    '''
                }
            }
        }
        stage('Apply Terraform') {
            steps {
                input message: "Approve Terraform Apply?", ok: "Deploy"
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsTest'
                ]]) {
                    sh '''

                    terraform apply -auto-approve tfplan
                    '''
                }
            }
        }

        stage('Optional Destroy') {
            steps {
                script {
                    def destroyChoice = input(
                        message: 'Do you want to run terraform destroy?',
                        ok: 'Submit',
                        parameters: [
                            choice(
                                name: 'DESTROY',
                                choices: ['no', 'yes'],
                                description: 'Select yes to destroy resources'
                            )
                        ]
                    )

                    if (destroyChoice == 'yes') {
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'aws-iam-user-creds'
                        ]]) {
                            sh 'terraform destroy -auto-approve'
                        }
                    } else {
                        echo "Skipping destroy"
                    }
                }
            }
        }
        
    }
    post {
        success {
            echo 'Terraform deployment completed successfully!'
        }
        failure {
            echo 'Terraform deployment failed!'
        }
    }
}