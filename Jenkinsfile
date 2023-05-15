pipeline {
    agent {
        label 'ansible-docker-agent'
    }

    parameters {
        string(name: 'ANSIBLE_HOST', defaultValue: '', description: 'Ansible Host')
    }

    stages {
        stage('Install requirements') {
            steps {
                script {
                    sh 'ansible-galaxy install -r requirements.yml'
                }
            }
        }

        stage('Create ssh key file') {
            steps {
                withCredentials([vaultString(credentialsId: 'vault-lightsail-key', variable: 'KEY')]) {
                    writeFile file: 'LightsailKey', text: "$KEY"
                }
                sh 'chmod 400 LightsailKey'
            }
        }

        stage('Create custom.yml') {
            steps {
                script {
                    String fileContents = """
                    username: ubuntu
                    ansible_ssh_private_key_file: LightsailKey
                    ansible_host: $params.ANSIBLE_HOST 
                    """
                    writeFile file: 'custom.yml', text: fileContents
                }
            }
        }

        stage('Create AWS credentials file') {
            steps {
                script {
                    withCredentials([[$class: 'VaultUsernamePasswordCredentialBinding', credentialsId: 'vault-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID']]) {
                        String fileContents = """
                        aws_access_key_id: ${AWS_ACCESS_KEY_ID}
                        aws_secret_access_key: ${AWS_SECRET_ACCESS_KEY}
                        duckdns_token: ${DUCKDNS_TOKEN}
                        """
                        writeFile file: 'secret.yml', text: fileContents
                    }
                }
            }
        }

        stage('Run command') {
            steps {
                sh 'ansible-playbook run.yml'
            }
        }
    }
}
