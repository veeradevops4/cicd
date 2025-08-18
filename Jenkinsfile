pipeline {
    agent any

    environment {
        // Set your environment variables or use Jenkins credentials plugin
        SFDX_CLIENT_ID = credentials('sfdx-client-id')
        SFDX_HUB_KEY = credentials('sfdx-server-key') // Use "Secret file" type in Jenkins
        SFDX_USERNAME = credentials('sfdx-username')
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Salesforce CLI') {
            steps {
                sh '''
                    wget https://developer.salesforce.com/media/salesforce-cli/sfdx-linux-amd64.tar.xz
                    mkdir sfdx-cli
                    tar xJf sfdx-linux-amd64.tar.xz -C sfdx-cli --strip-components 1
                    ./sfdx-cli/install
                    export PATH=./sfdx-cli/bin:$PATH
                    sfdx --version
                '''
            }
        }

        stage('Authenticate to Salesforce') {
            steps {
                sh '''
                    echo "$SFDX_HUB_KEY" > server.key
                    sfdx auth:jwt:grant \
                        --clientid $SFDX_CLIENT_ID \
                        --jwtkeyfile server.key \
                        --username $SFDX_USERNAME \
                        --instanceurl https://login.salesforce.com \
                        --setdefaultdevhubusername
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    sfdx force:apex:test:run \
                        --resultformat human \
                        --outputdir test-results \
                        --codecoverage \
                        --wait 10
                '''
            }
        }

        stage('Deploy to Org') {
            steps {
                sh '''
                    sfdx force:source:deploy \
                        -p force-app \
                        -u $SFDX_USERNAME \
                        --testlevel RunLocalTests
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/test-result.xml'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
