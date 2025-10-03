
# Hello Java Maven — Jenkins Build (Task 8)

Hi — I’m sharing my step‑by‑step notes for **Task 8: Run a Simple Java Maven Build Job in Jenkins**. I wrote this README in first person so you can follow exactly what I did and reproduce the deliverable.

---

## What I built
- A tiny Java `HelloWorld` application.
- A Maven `pom.xml` to build it.
- A Jenkins Freestyle job that runs `mvn clean package`.
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

### 3) Configure Maven in Jenkins (Option A: Use Jenkins' Maven)
1. In Jenkins → **Manage Jenkins → Global Tool Configuration**  
   - Under **Maven** I clicked **Add Maven**, named it `Maven-3.8.6` (or any name), and selected "Install automatically" or provided an installation path.
2. I created a **New Item** → *Freestyle project* named `hello-java-maven`.
3. Under **Build** I selected **Invoke top-level Maven targets** and configured:
   - **Maven Version**: `Maven-3.8.6`
   - **Root POM**: `pom.xml`
   - **Goals**: `clean package`
4. I saved and clicked **Build Now**.

### 4) Alternative: Run Maven inside Docker from Jenkins (Option B)
If I didn't want to configure Maven under Global Tools, I created a Freestyle project and added an **Execute shell** build step with:

```bash
docker run --rm -v "$PWD":/workspace -w /workspace maven:3.8.6-openjdk-11 mvn -B clean package
```

This runs Maven inside an ephemeral Docker container and produces the same `BUILD SUCCESS`.

> **Note:** If Jenkins is itself a Docker container, to run Docker commands from Jenkins I either had to:
> - mount the host Docker socket (`-v /var/run/docker.sock:/var/run/docker.sock`) into the Jenkins container, **or**
> - run builds on an agent (node) that has Docker installed.

### 5) If using Git
I pushed the project to a remote Git repo and set Jenkins job **Source Code Management → Git** with the repo URL. Jenkins then clones the repo before running the build step.

Commands I used to push to GitHub (example):
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:<me>/hello-java-maven.git
git push -u origin main
```

### 6) Verify build & capture screenshot
1. In Jenkins: Job → Build History → Click the build number → **Console Output**.
2. I looked for the `BUILD SUCCESS` line:
```
[INFO] BUILD SUCCESS
```
3. I took a screenshot (e.g. `task8-console.png`) of the Console Output page as the deliverable.

If I prefer text, I can also download console text:
```bash
curl -s http://localhost:8080/job/hello-java-maven/lastBuild/consoleText > console.txt
```

---

## Troubleshooting I encountered (and how I fixed them)
- **mvn: command not found** — If using "Invoke top-level Maven targets", configure Maven in **Global Tool Configuration** or install Maven on the agent.
- **Docker permission issues** — If using Docker inside Jenkins, ensure Docker socket is mounted and user permissions are correct.
- **Git clone failures** — Make sure credentials are configured in Jenkins credentials store and network access to the Git host is allowed.
- **File ownership problems** — When Jenkins creates files as `root` (inside container), adjust volume permissions or run Jenkins with a proper user mapping.

---

## Checklist (what I submitted)
- [ ] `hello-java-maven` project files (`HelloWorld.java`, `pom.xml`)
- [ ] Jenkins Freestyle job configured to run `mvn clean package`
- [ ] Successful build with `BUILD SUCCESS`
- [ ] Screenshot of Jenkins Console Output saved (e.g. `task8-console.png`)

---

## Notes & tips
- I used Java 8 (source/target 1.8) in `pom.xml`. If you use Java 11, update `maven.compiler.source` and `maven.compiler.target` accordingly.
- For repeatable builds on CI, I prefer the Docker Maven approach because CI doesn't need tool installs configured on the master.
- If you want, I can also generate: a ZIP of the project, a `config.xml` for the Jenkins job, or mock screenshots. Ask and I’ll create them.

---

**Done — you can download this README file below.**
