# Hello Java Maven - Jenkins Build (Task 8)

---

## What I built
- A tiny Java `HelloWorld` application.
- A Maven `pom.xml` to build it.
- A Jenkins Maven job that runs `mvn clean package`.
- I captured the Jenkins Console Output (where you should see `BUILD SUCCESS`).

---

## Project structure
```
hello-java-maven/
├─ pom.xml
└─ src/
   └─ main/
      └─ java/
         └─ HelloWorld.java
```

### `src/main/java/HelloWorld.java`
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Jenkins + Maven!");
    }
}
```

### `pom.xml`
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>hello</artifactId>
  <version>1.0</version>

  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
          <source>${maven.compiler.source}</source>
          <target>${maven.compiler.target}</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

---

## Step‑by‑step: How I ran this locally and in Jenkins

### 1) Create the project locally
1. I created the folder structure as shown above.
2. I saved `HelloWorld.java` and `pom.xml` files with the content above.

Commands I ran to quickly test locally (optional):
```bash
cd hello-java-maven
mvn -B clean package
# Expect: BUILD SUCCESS and target/hello-1.0.jar
```
<img width="1920" height="1080" alt="Screenshot from 2025-10-03 20-15-25" src="https://github.com/user-attachments/assets/c224e078-63af-4cbd-ae4d-f96c4811ac91" />
<img width="1920" height="1080" alt="Screenshot from 2025-10-03 20-15-29" src="https://github.com/user-attachments/assets/7110275f-edb3-492b-bb3f-a0eda00d8f04" />

---

### 2) Start Jenkins (I used Docker)
I started Jenkins LTS in Docker for a quick, local Jenkins server:

```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

To get the initial admin password (first time setup):
```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

I opened `http://localhost:8080` in my browser and completed the setup wizard (installed suggested plugins).

---

### 3) Configure Maven in Jenkins (Option A: Use Jenkins' Maven)
1. In Jenkins → **Manage Jenkins → Global Tool Configuration**  
   - Under **Maven** I clicked **Add Maven**, named it `Maven-3.8.6` (or any name), and selected "Install automatically" or provided an installation path.
2. I created a **New Item** → *Maven project* named `hello-java-maven`.
3. Under **Build** I selected **Invoke top-level Maven targets** and configured:
   - **Root POM**: `pom.xml`
   - **Goals**: `clean install`
4. I saved and clicked **Build Now**.

<img width="1920" height="1080" alt="Screenshot from 2025-10-03 20-19-11" src="https://github.com/user-attachments/assets/482a254d-79c9-42cc-85dc-cf30ac1aeb58" />

---

### 4) If using Git
I pushed the project to a remote Git repo and set Jenkins job **Source Code Management → Git** with the repo URL. Jenkins then clones the repo before running the build step.

Commands I used to push to GitHub (example):
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin < repositry link >
git push -u origin main
```

<img width="1920" height="1080" alt="Screenshot from 2025-10-03 18-39-14" src="https://github.com/user-attachments/assets/96707414-a5f3-48c2-930d-56f4586d6d9d" />

---

### 5) Verify build & capture screenshot
1. In Jenkins: Job → Build History → Click the build number → **Console Output**.
2. I looked for the `BUILD SUCCESS` line:
```
[INFO] BUILD SUCCESS
```
<img width="1920" height="1080" alt="Screenshot from 2025-10-03 20-13-30" src="https://github.com/user-attachments/assets/29678289-083a-4e07-9418-1a9c591dc8a6" />
<img width="1920" height="1080" alt="Screenshot from 2025-10-03 20-20-40" src="https://github.com/user-attachments/assets/53a70392-42ee-4f56-b7a3-54456037fec4" />

---

## Troubleshooting I encountered (and how I fixed them)
- **mvn: command not found** - If using "Invoke top-level Maven targets", configure Maven in **Global Tool Configuration** or install Maven on the agent.
- **Docker permission issues** - If using Docker inside Jenkins, ensure Docker socket is mounted and user permissions are correct.
- **Git clone failures** - Make sure credentials are configured in Jenkins credentials store and network access to the Git host is allowed.
- **File ownership problems** - When Jenkins creates files as `root` (inside container), adjust volume permissions or run Jenkins with a proper user mapping.

---
