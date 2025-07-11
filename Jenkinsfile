pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    // environment { 
    //     packageVersion = ''
    //     nexusURL = '172.31.5.95:8081'
    // }

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    parameters {
        string(name: 'version', defaultValue: '', description: 'What is the artifact version?')
        string(name: 'environment', defaultValue: 'dev', description: 'What is environment?')
        booleanParam(name: 'Destroy', defaultValue: false, description: 'What is Destroy?')
        booleanParam(name: 'Create', defaultValue: false, description: 'What is Create?')
    }

    // build
    stages {
        stage('Print version') {
            steps {
                sh """
                    echo "version: ${params.version}"
                    echo "environment: ${params.environment}"
                    echo "Destroy: ${params.Destroy}"
                    echo "Create: ${params.Create}"
                """
            }
        }

        stage('Init') {
            steps {
                sh """
                    cd terraform
                    terraform init --backend-config=${params.environment}/backend.tf -upgrade -reconfigure
                """
            }
        }

        stage('Plan') {
            when {
                expression { return params.Create }
            }
            steps {
                sh """
                    cd terraform
                    terraform plan -var-file=${params.environment}/${params.environment}.tfvars -var="app_version=${params.version}"
                """
            }
        }

        stage('Apply') {
            when {
                expression { return params.Create }
            }
            steps {
                sh """
                    cd terraform
                    terraform apply -var-file=${params.environment}/${params.environment}.tfvars -var="app_version=${params.version}" -auto-approve
                """
            }
        }

stage('Destroy') {
    when {
        expression { return params.Destroy }
    }
    steps {
        sh """
            cd terraform
            terraform destroy -var-file=${params.environment}/${params.environment}.tfvars -var="app_version=${params.version}" -auto-approve
        """
    }
}

        // stage('EKS Login') {
        //     steps {
        //         script {
        //             sh """
        //                 aws eks update-kubeconfig --region us-east-1 --name roboshop-${params.environment}
        //             """
        //         }
        //     }
        // }

        // stage('EKS Deploy') {
        //     steps {
        //         script {
        //             sh """
        //                 cd helm
        //                 sed -i 's/IMAGE_VERSION/${params.version}/g' values.yaml
        //                 helm upgrade --install catalogue -n roboshop .
        //             """
        //         }
        //     }
        // }
    }

    // post build
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure { 
            echo 'this runs when pipeline is failed, used generally to send some alerts'
        }
        success {
            echo 'I will say Hello when pipeline is success'
        }
    }
}
