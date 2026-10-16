
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        REPO_URL = 'https://github.com/cmmani/java_simple.git'
        TARGET_HOST = '192.168.1.117'
        WAR_NAME = 'java-sample-app-1.0.0.war'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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

        stage('Upload and Deploy') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'target-developer-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    script {
                        def remote = [
                            name: 'tomcat-target',
                            host: env.TARGET_HOST,
                            user: SSH_USER,
                            identityFile: SSH_KEY,
                            allowAnyHosts: true
                        ]

                        sshPut remote: remote,
                            from: env.WAR_NAME,
                            into: '/tmp'

                        sshCommand remote: remote,
                            command: '''
                                set -eu

                                SRC=/tmp/java-sample-app-1.0.0.war
                                DST=/opt/tomcat/webapps/java-sample-app-1.0.0.war
                                BACKUP=/opt/tomcat/war-backups

                                sudo -n /bin/bash -c '
                                    set -eu

                                    test -s /tmp/java-sample-app-1.0.0.war
                                    /usr/bin/jar tf /tmp/java-sample-app-1.0.0.war >/dev/null

                                    mkdir -p /opt/tomcat/war-backups

                                    if [ -f /opt/tomcat/webapps/java-sample-app-1.0.0.war ]; then
                                        cp -p /opt/tomcat/webapps/java-sample-app-1.0.0.war \
                                        "/opt/tomcat/war-backups/java-sample-app-1.0.0-$(date +%Y%m%d-%H%M%S).war"
                                    fi

                                    install -o root -g root -m 0644 \
                                    /tmp/java-sample-app-1.0.0.war \
                                    /opt/tomcat/webapps/java-sample-app-1.0.0.war.new

                                    mv -f /opt/tomcat/webapps/java-sample-app-1.0.0.war.new \
                                    /opt/tomcat/webapps/java-sample-app-1.0.0.war

                                    echo "WAR deployment successful"
                                '
                            '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: WAR uploaded and deployment command completed.'
        }
        failure {
            echo 'FAILED: Check Console Output for the exact error.'
        }
    }
}
