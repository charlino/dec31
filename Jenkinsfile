pipeline {
    agent any
    environment {
        SSH_CRED = credentials('server-key')
        def CONNECT = 'ssh -o StrictHostKeyChecking=no ubuntu@15.222.255.218'
        NEXUS_URL = 'http://3.99.185.223:8081/repository/workshop-app/'
        NEXUS_CRED = credentials('artifact') // Add your Nexus credentials in Jenkins
        ARTIFACT_GROUP = 'com/example'
        ARTIFACT_ID = 'webapp'
        ARTIFACT_VERSION = '1.0.0'
        ARTIFACT = "${ARTIFACT_ID}-${ARTIFACT_VERSION}.zip"
    }
    stages {
        
        stage('Build') {
            steps {
                echo 'Building the app'
                sh "pwd"
                sh "ls"
                sh "zip -r ${ARTIFACT} ."
                sh "ls"
            }
        }
        
        stage('Upload to Nexus') {
            steps {
                echo 'Uploading artifact to Nexus'
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh """
                        curl -u $NEXUS_USER:$NEXUS_PASSWORD --upload-file ${ARTIFACT} \
                        ${NEXUS_URL}/${ARTIFACT_GROUP}/${ARTIFACT_ID}/${ARTIFACT_VERSION}/${ARTIFACT}
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying the app'
                sshagent(['server-key']) {
                    // Download the artifact from Nexus
                    sh "curl -u $NEXUS_USER:$NEXUS_PASSWORD -O \
                    ${NEXUS_URL}/${ARTIFACT_GROUP}/${ARTIFACT_ID}/${ARTIFACT_VERSION}/${ARTIFACT}"
                    
                    // Deploy the artifact to the target server
                    sh 'scp -o StrictHostKeyChecking=no -i $SSH_CRED ${ARTIFACT} ubuntu@15.222.255.218:/home/ubuntu'
                    sh '$CONNECT "sudo apt install zip -y"'
                    sh '$CONNECT "sudo rm -rf /var/www/html/"'
                    sh '$CONNECT "sudo mkdir /var/www/html/"'
                    sh '$CONNECT "sudo unzip /home/ubuntu/${ARTIFACT} -d /var/www/html/"'
                }
            }
        }

        stage('Clean-Up') {
            steps {
                echo 'Cleaning up workspace'
                deleteDir()
            }
        }
    }
    post {
        failure {
            echo 'Build failed. Cleaning up workspace...'
            cleanWs()
        }
    }
}