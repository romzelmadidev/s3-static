
pipeline {
    agent any

    environment {
        AZURE_STORAGE_ACCOUNT = 'pocstoreeastus7683'
        AZURE_FILE_SHARE     = 'webcontent'
        STAGING_URL          = 'http://poc-staging-zel7683.eastus.azurecontainer.io'
        EXPECTED_CONTENT     = 'Welcome'

        PROD_EC2_1 = '10.0.10.112'  // Replace with EC2 #1 Private/Public IP
        PROD_EC2_2 = '10.0.20.148'  // Replace with EC2 #2 Private/Public IP
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Staging (Azure)') {
            steps {
                withCredentials([string(credentialsId: 'AZURE_STORAGE_KEY', variable: 'STORAGE_KEY')]) {
                    sh '''
                        az storage file upload \
                          --account-name ${AZURE_STORAGE_ACCOUNT} \
                          --account-key "${STORAGE_KEY}" \
                          --share-name ${AZURE_FILE_SHARE} \
                          --source index.html \
                          --path index.html
                    '''
                }
            }
        }

        stage('Test Staging') {
            steps {
                sh '''
                    HTTP_STATUS=$(curl -o /tmp/staging_response.txt -s -w "%{http_code}" ${STAGING_URL})
                    if [ "$HTTP_STATUS" -ne 200 ]; then
                        echo "ERROR: Staging returned status $HTTP_STATUS"
                        exit 1
                    fi

                    if grep -q "${EXPECTED_CONTENT}" /tmp/staging_response.txt; then
                        echo "SUCCESS: Staging content assertion passed!"
                    else
                        echo "ERROR: Expected content '${EXPECTED_CONTENT}' not found!"
                        exit 1
                    fi
                '''
            }
        }

       stage('Deploy to Production (AWS)') {
    steps {
        sshagent(['EC2_SSH_KEY']) { 
            sh '''
            scp -o StrictHostKeyChecking=no index.html ubuntu@${PROD_EC2_1}:/tmp/index.html
            ssh -o StrictHostKeyChecking=no ubuntu@${PROD_EC2_1} 'sudo mv /tmp/index.html /var/www/html/index.html'
            '''
        }
    }
}
    }
}
EOF
