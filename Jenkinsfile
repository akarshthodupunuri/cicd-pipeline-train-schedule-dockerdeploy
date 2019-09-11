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
        stage('building the docker image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("akarshthodupunuri/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push the docker image to registry') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_creds') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_creds', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_creds') {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull akarshthodupunuri/train-schedule:${env.BUILD_NUMBER}\""
                            try {
                                sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                                sh "sshpass -p '$USERPASS' -v ssh -o StricthostKeychecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                            } catch (err) {
                                echo: 'caught error: $err'
                            }
                            sh "sshpass -p $USERPASS -v ssh -o StrictHostkeyChecking=no $USERNAME@$prod_ip \"docker run --restart=always --name train-schedule -p 8080:8080 -d akarshthodupunuri/train-schedule:${env.BUILD_NUMBER}\""
                        }

                    }
                }

            }
        }
    }
}
