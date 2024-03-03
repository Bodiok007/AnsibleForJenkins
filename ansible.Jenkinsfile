pipeline {
    agent any
    
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION    = 'us-east-1'
        INSTANCE_TAG          = 'app-instance'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[url: 'https://github.com/Bodiok007/AnsibleForJenkins.git']]
                ])
            }
        }
        
        stage('GenerateInventoryAndApply') {
            steps {
                script {

                    withCredentials([sshUserPrivateKey(credentialsId: 'devops-ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'USER')]) {
                        def instanceIp = getPublicIpAddress('app-instance')
                        echo "Instance IP: ${instanceIp}"
                        
                        def inventoryContent = "[app]\n${instanceIp} ansible_ssh_extra_args='-o StrictHostKeyChecking=no' ansible_ssh_user='${USER}' ansible_ssh_private_key_file='${SSH_KEY}'"
                        writeFile file: "inventory.ini", text: inventoryContent
                        
                        sh '''
                            ansible-playbook playbook.yml -i inventory.ini
                        '''
                    }
                    
                    sh '''
                            cat inventory.ini
                        '''
                }
            }
        }
    }
}

def getPublicIpAddress(tag) {
    def instanceIp = sh(script: """
        aws ec2 describe-instances \
        --region ${AWS_DEFAULT_REGION} \
        --query 'Reservations[].Instances[?Tags[?Key==`Name` && Value==`app-instance`]].PublicIpAddress' \
        --output text
    """, returnStdout: true).trim()

    return instanceIp
}
