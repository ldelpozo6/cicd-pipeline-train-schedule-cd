pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '''
                                            sudo yum install -y unzip &&\
                                            sudo rm -rf /opt/train-schedule &&\
                                            sudo unzip /tmp/trainSchedule.zip -d /opt/train-schedule &&\
                                            sudo cp -f /opt/train-schedule/systemd/system/train-schedule.service /lib/systemd/system/ &&\
                                            sudo systemctl daemon-reload &&\
                                            sudo systemctl enable train-schedule.service &&\
                                            sudo systemctl restart train-schedule
                                            '''
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '''
                                            sudo yum install -y unzip &&\
                                            sudo rm -rf /opt/train-schedule &&\
                                            sudo unzip /tmp/trainSchedule.zip -d /opt/train-schedule &&\
                                            sudo cp -f /opt/train-schedule/systemd/system/train-schedule.service /lib/systemd/system/ &&\
                                            sudo systemctl daemon-reload &&\
                                            sudo systemctl enable train-schedule.service &&\
                                            sudo systemctl restart train-schedule
                                            '''
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
