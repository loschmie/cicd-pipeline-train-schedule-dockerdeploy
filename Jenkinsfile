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
        stage("Build docker image") {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build('loschmie/trainschedule')
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage("Push Docker Image") {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry("https://registry.hub.docker.com", 'dockerhub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage("Deploy to production") {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production!'
                milestone(1)
                withCredentials ([usernamePassword(credentials: 'webserver_login', usernameVariable: "USERNAME", passwordVariable: "USERPASS")])
                    script {
                        sh "shpass -p '$USERPAS' -v ssh -o StictHostKeyChecking=no $USERNAME=@$prod_ip \"docker pull loschmie/train-schedule:${env.BUILD_NUMBER}\""
                        try { 
                            sh "shpass -p '$USERPAS' -v ssh -o StictHostKeyChecking=no $USERNAME=@$prod_ip \"docker stop train-schedule\""
                            sh "shpass -p '$USERPAS' -v ssh -o StictHostKeyChecking=no $USERNAME=@$prod_ip \"docker rm train-schedule\""
                        }catch (err) {
                            echo 'caught error: $err'
                        }

                    }sh "shpass -p '$USERPAS' -v ssh -o StictHostKeyChecking=no $USERNAME=@$prod_ip \"docker run --restart always --name train-schedule -p 80:80 -d loschimie/train-schedule:${env.$BUILD_NUMBER}\""
            }
        }
    }
}