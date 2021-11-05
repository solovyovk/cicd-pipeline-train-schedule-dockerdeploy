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
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("ksolovyov/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image to DockerHub') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_ks') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Production'){
            when {
                branch 'master'
            }
            steps {
                input 'Ready to deploy on Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'deploy_usr', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyCheking=no $USERNAME@$prod_ip \"docker pull ksolovyov/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyCheking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyCheking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught erro: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyCheking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d ksolovyov/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}