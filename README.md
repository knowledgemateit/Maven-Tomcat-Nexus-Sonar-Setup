Sonarqube : Code quality and check tool > 9000

Nexus: Artifactory tool > 8081

Jenkins : CI/CD > 8080

Tomcat : 8080

mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=f5782cfeafd95d3216f992b6c35bbdfe5fd67ac3

Nexus:

vi /etc/maven/settings.xml

    <server>
    
      <id>deploymentRepo</id>
      
      <username>admin</username>
      
      <password>1d7edc68-4300-43d0-b648-assc7d4c7049fd</password>
      
    </server>

mvn clean deploy


apt install docker.io -y

systemctl start docker

docker run -d --name tomcat -p 8090:8080 consol/tomcat-7.0

docker cp target/java-tomcat-maven-example.war tomcat:/opt/tomcat/webapps/



tomcat username & password is admin / admin
