mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=f5782cfeafd95d3216f992b6c35bbdfe5fd67ac3

mvn clean deploy

#mvn install tomcat7:deploy

apt install docker.io

systemctl start docker

docker run -d --name tomcat -p 8090:8080 consol/tomcat-7.0
