# 🗳️ PollApp — Java Web Application CI/CD Pipeline

A Java 17 web application built with Apache Tomcat 10.1, featuring integrated code quality analysis via SonarQube and artifact management via Nexus Repository Manager — all hosted on Amazon Linux.

---

## 🛠️ Project Stack

| Tool | Version | Purpose |
|---|---|---|
| Java | 17 (Amazon Corretto) | Runtime & Development |
| Apache Tomcat | 10.1 | Servlet Container |
| Maven | Latest | Build Tool |
| SonarQube | LTS Community | Static Code Analysis |
| Nexus Repository Manager | 3.x | Artifact Storage |
| Docker | Latest | Containerization for Tools |

---

## 📂 Project Structure

```
PollApp/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── PollServlet.java       # Core voting logic
│   │   │   └── ResultsServlet.java    # Data API for results
│   │   └── webapp/
│   │       └── index.jsp              # Frontend with AJAX polling
│   └── test/
│       └── java/
│           └── PollTest.java          # JUnit 5 unit tests
└── pom.xml
```

---

## ⚙️ Environment Setup (Amazon Linux)

Tomcat: Web Server
git,java,Maven,Docker:

```bash
sudo dnf update -y
sudo dnf install java-17-amazon-corretto-devel maven git -y

sudo dnf install docker -y
systemctl start docker 
systemctl enable docker
```

```bash
sudo docker run -d -p 8080:8080 --name tomcat-prod \
  --memory="512m" \
  -e JAVA_OPTS="-Xms256m -Xmx256m" \
  tomcat:10.1-jdk17-corretto
```

```bash
cat <<EOF > tomcat-users.xml
<tomcat-users>
  <role rolename="manager-script"/>
  <role rolename="manager-gui"/>
  <role rolename="admin-gui"/>
  <user username="admin" password="admin123" roles="manager-gui,admin-gui,manager-script"/>
</tomcat-users>
EOF
```

# Copy it into the container
```bash
sudo docker cp tomcat-users.xml tomcat-prod:/usr/local/tomcat/conf/tomcat-users.xml
sudo docker exec -it tomcat-prod mv /usr/local/tomcat/webapps.dist/manager /usr/local/tomcat/webapps/
sudo docker exec -it tomcat-prod mv /usr/local/tomcat/webapps.dist/host-manager /usr/local/tomcat/webapps/
sudo docker exec -it tomcat-prod mv /usr/local/tomcat/webapps.dist/ROOT /usr/local/tomcat/webapps/
```

```bash
cat <<EOF > context.xml
<?xml version="1.0" encoding="UTF-8"?>
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
EOF
```

```bash
sudo docker cp context.xml tomcat-prod:/usr/local/tomcat/webapps/manager/META-INF/context.xml
sudo docker cp context.xml tomcat-prod:/usr/local/tomcat/webapps/host-manager/META-INF/context.xml
```

```bash
sudo docker restart tomcat-prod
```

```bash
git clone https://github.com/knowledgemateit/Project-Maven-Tomcat-Nexus-Sonar-Setup-PollApp.git
```
```bash
mvn clean package
```

```bash
sudo docker cp target/PollApp.war tomcat-prod:/usr/local/tomcat/webapps
```


## 🔍 SonarQube: Code Quality Analysis

SonarQube is deployed via Docker and analyzes code for bugs, vulnerabilities, and code smells.

### Deployment

```bash
sudo sysctl -w vm.max_map_count=262144
sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```

Access the dashboard at: `http://<EC2-IP>:9000`

### Running Analysis

```bash
export MAVEN_OPTS="-Xmx256m"
mvn sonar:sonar \
  -Dsonar.projectKey=PollApp \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your_generated_token
```

> 💡 Generate your token from **SonarQube → My Account → Security**.

---

## 📦 Nexus: Artifact Management

Nexus Repository Manager stores versioned build artifacts for deployment and rollback.

### Deployment

```bash
sudo docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

Access the dashboard at: `http://<EC2-IP>:8081`

### Retrieve Initial Admin Password

```bash
sudo docker exec -it nexus cat /nexus-data/admin.password
```

### Maven Integration

Add Nexus credentials to `~/.m2/settings.xml`:

```xml
<servers>
  <server>
    <id>nexus-snapshots</id>
    <username>admin</username>
    <password>your_password</password>
  </server>
</servers>
```

### Deploy Artifact to Nexus

```bash
mvn clean deploy -DskipTests
```

---

## 🚦 How to Run

### Build & Test
```bash
mvn clean verify
```

### Analyze Code Quality
```bash
mvn sonar:sonar
```

### Upload Artifact to Nexus
```bash
mvn deploy
```

### Access the Application
```
http://<EC2-IP>:8080/PollApp
```

---

## 🔌 Port Reference

| Service | Port |
|---|---|
| Apache Tomcat | 8080 |
| SonarQube | 9000 |
| Nexus Repository | 8081 |

---

## ⚠️ Notes

- All tool containers (SonarQube, Nexus) are managed by Docker and must be running before analysis or deployment.
- The `MAVEN_OPTS="-Xmx256m"` flag is required on low-memory instances to prevent out-of-memory errors during Maven builds.
- Swap space is persistent across reboots due to the `/etc/fstab` entry.
