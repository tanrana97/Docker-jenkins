
AWS, Jenkins Pipeline, Dockerfile, Docker

pipeline {
agent any
environment {
PATH="/project/build/apache-maven-3.8.6/bin:$PATH"
}
stages {
stage ("clone") {
steps {
dir ("/project/code") {
sh "rm -rf /project/code/*"
sh "git clone https://github.com/tanrana97/game-of-life.git"
}
}
}
stage ("build") {
steps {
dir ("/project/code/game-of-life") {
sh "mvn clean package"
}
}
}
stage ("copy_s3") {
steps {
sh 'aws s3 cp /project/code/game-of-life/gameoflife-web/target/gameoflife.war s3://dockdeploy'
}
}
stage ("deploy") {
steps {
dir ('/project/docker'){
sh 'aws s3 cp s3://dockdeploy/gameoflife.war /project/docker/code'
sh 'sudo docker build -t tomcat:1.0 .'
sh 'sudo docker run -d -itp 8081:8080 -v /project/docker/code:/usr/local/tomcat/webapps tomcat:1.0'
sh 'sudo docker run -d -itp 8082:8080 -v /project/docker/code:/usr/local/tomcat/webapps tomcat:1.0'
}
}
}
}
}
