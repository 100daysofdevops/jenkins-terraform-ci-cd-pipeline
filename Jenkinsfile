pipeline {
    agent any

    tools {
        "org.jenkinsci.plugins.terraform.TerraformInstallation" "terraform"
    }

    parameters {
        string(name: 'WORKSPACE', defaultValue: 'development', description: 'setting up workspace for terraform')
    }

    environment {
        TF_HOME = tool('terraform')
        TP_LOG = "WARN"
        PATH = "$TF_HOME:$PATH"
        ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID')
        SECRET_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        dir('jenkins-terraform-pipeline/ec2_pipeline/') {

            stage('TerraformInit') {
                steps {
                    sh """
                        terraform init -input=false
                    """
                }
            }

            stage('TerraformFormat') {
                steps {
                    sh "terraform fmt -list=true -write=false -diff=true -check=true"
                }
            }

            stage('TerraformValidate') {
                steps {
                    sh "terraform validate"
                }
            }

            stage('TerraformPlan') {
                steps {
                    script {
                        try {
                            sh "terraform workspace new ${params.WORKSPACE}"
                        } catch (Exception err) {
                            sh "terraform workspace select ${params.WORKSPACE}"
                        }
                        sh "terraform plan -var 'access_key=$ACCESS_KEY' -var 'secret_key=$SECRET_KEY' -out terraform.tfplan"
                        stash name: "terraform-plan", includes: "terraform.tfplan"
                    }
                }
            }

            stage('TerraformApply') {
                steps {
                    script {
                        def shouldApply = false
                        try {
                            input message: 'Can you please confirm the apply', ok: 'Ready to Apply the Config'
                            shouldApply = true
                        } catch (Exception err) {
                            shouldApply = false
                            currentBuild.result = 'UNSTABLE'
                        }

                        if (shouldApply) {
                            unstash "terraform-plan"
                            sh 'terraform apply terraform.tfplan'
                        }
                    }
                }
            }
        }
    }
}
