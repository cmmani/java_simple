
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        REPO_URL = 'https://github.com/cmmani/java_simple.git'
        BRANCH = 'main'

        TARGET_HOST = '192.168.1.117'
        TARGET_USER = 'developer'

        WAR_NAME = 'java-sample-app-1.0.0.war'
        REMOTE_TMP = '/tmp/java-sample-app-1.0.0.war'
        DEPLOY_DIR = '/opt/tomcat/webapps'
        TARGET_WAR = '/opt/tomcat/webapps/java-sample-app-1.0.0.war'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}",
                    url: "${REPO_URL}"
            }
        }

        stage('Validate WAR') {
            steps {
                sh '''
                    set -eu
                    test -s "$WAR_NAME"
                    echo "WAR file found:"
                    ls -lh "$WAR_NAME"
                '''
            }
        }

        stage('Upload WAR') {
            steps {
                sshagent(credentials: ['target-developer-ssh']) {
                    sh '''
                        set -eu

                        ssh -o BatchMode=yes \
                            "${TARGET_USER}@${TARGET_HOST}" \
                            "test -d /opt/tomcat/webapps"

                        scp -o BatchMode=yes \
                            "$WAR_NAME" \
                            "${TARGET_USER}@${TARGET_HOST}:${REMOTE_TMP}"

                        echo "WAR uploaded successfully"
                    '''
                }
            }
        }

        stage('Backup and Deploy') {
            steps {
                sshagent(credentials: ['target-developer-ssh']) {
                    sh '''
                        set -eu

                        ssh -o BatchMode=yes \
                            "${TARGET_USER}@${TARGET_HOST}" \
                            "sudo -n /bin/bash -c 'set -eu
                            SRC=/tmp/java-sample-app-1.0.0.war
                            DST=/opt/tomcat/webapps/java-sample-app-1.0.0.war
                            BACKUP=/opt/tomcat/war-backups

                            test -s \\"\\$SRC\\"
                            /usr/bin/jar tf \\"\\$SRC\\" >/dev/null
                            mkdir -p \\"\\$BACKUP\\"

                            if [ -f \\"\\$DST\\" ]; then
                                cp -p \\"\\$DST\\" \\"\\$BACKUP/java-sample-app-1.0.0-\\$(date +%Y%m%d-%H%M%S).war\\"
                            fi

                            install -o root -g root -m 0644 \\"\\$SRC\\" \\"\\$DST.new\\"
                            mv -f \\"\\$DST.new\\" \\"\\$DST\\"

                            echo Deployment successful
                            '"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: WAR deployed to 192.168.1.117.'
        }
        failure {
            echo 'FAILED: Check the Jenkins Console Output.'
        }
    }
}
