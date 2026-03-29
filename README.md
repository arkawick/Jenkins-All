# Jenkins-All

Local Jenkins setup with Jenkins Job Builder (JJB) for managing jobs as code.

## Stack

- **Jenkins** — running via Docker on port `8090`
- **Jenkins Job Builder** — Python tool to define Jenkins jobs in YAML

## Prerequisites

- Docker
- Python 3.x + pip

## Setup

### 1. Start Jenkins

```bash
docker run -d --name jenkins \
  -p 8090:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

> **Note:** Port 8090 is used instead of the default 8080 to avoid conflicts with other services like Apache Tomcat that also default to 8080.

Open http://localhost:8090 and complete the setup wizard.

### 2. Install Jenkins Job Builder

```bash
pip install jenkins-job-builder
```

### 3. Configure credentials

Edit `jenkins.ini` with your Jenkins URL and credentials:

```ini
[jenkins]
url=http://localhost:8090
user=your-username
password=your-password-or-api-token
```

> **Note:** Use an API token instead of your password for better security.
> Generate one at: Jenkins → Your User → Configure → API Token.

---

## JJB Usage

### Test jobs locally (no Jenkins needed)

```bash
jenkins-jobs test jobs/
```

Prints the generated XML without touching Jenkins.

### Push jobs to Jenkins

```bash
jenkins-jobs --conf jenkins.ini update jobs/
```

### Delete a job from Jenkins

```bash
jenkins-jobs --conf jenkins.ini delete <job-name>
```

> **Note:** `jenkins-jobs get-jobs` was removed in JJB 6.x. Use `curl.exe` to fetch job configs instead (see below).

---

## Exporting Job Configs (XML)

You can export a job's configuration as XML and use it to clone or migrate jobs.

### Fetch a job's config.xml

```powershell
curl.exe -u user:password http://localhost:8090/job/hello-world/config.xml -o hello-world-config.xml
```

### Create a new job from XML

```powershell
curl.exe -u user:password -X POST "http://localhost:8090/createItem?name=hello-world-copy" `
  -H "Content-Type: application/xml" `
  -d @hello-world-config.xml
```

### Update an existing job from XML

```powershell
curl.exe -u user:password -X POST "http://localhost:8090/job/hello-world/config.xml" `
  -H "Content-Type: application/xml" `
  -d @hello-world-config.xml
```

> **Note:** On Windows, always use `curl.exe` not `curl` — PowerShell aliases `curl` to `Invoke-WebRequest` which has different syntax.

### JJB YAML vs raw config.xml

| | JJB YAML | Raw config.xml |
|---|---|---|
| Human readable | Yes | Not really |
| Version control friendly | Yes | Possible but messy |
| Templating / reuse | Yes | No |
| Captures exact job state | No | Yes |

---

## Exporting Build Logs

Build logs are plain text files stored per build inside Jenkins home.

### Via API

```powershell
# Specific build
curl.exe -u user:password http://localhost:8090/job/hello-world/1/consoleText -o build-1.log

# Latest build
curl.exe -u user:password http://localhost:8090/job/hello-world/lastBuild/consoleText
```

### Via docker cp

```powershell
# Single build log
docker cp jenkins:/var/jenkins_home/jobs/hello-world/builds/1/log ./build-1.log

# All logs for a job
docker cp jenkins:/var/jenkins_home/jobs/hello-world/builds ./hello-world-logs
```

### View log directly in terminal

```powershell
docker exec jenkins sh -c "cat /var/jenkins_home/jobs/hello-world/builds/lastSuccessfulBuild/log"
```

---

## Backup & Restore

### Option 1 — Full backup (recommended)

Backs up everything: job configs, build history, plugin configs, user data.

```powershell
docker run --rm -v jenkins_home:/data -v C:/Users/Arkajyoti/Downloads/Jenkins-All:/backup alpine `
  tar czf /backup/jenkins-full-backup.tar.gz -C /data .
```

**Restore:**

```powershell
docker run --rm -v jenkins_home:/data -v C:/Users/Arkajyoti/Downloads/Jenkins-All:/backup alpine `
  tar xzf /backup/jenkins-full-backup.tar.gz -C /data
```

### Option 2 — Export job configs only (no build history)

```powershell
# Single job
curl.exe -u user:password http://localhost:8090/job/hello-world/config.xml -o hello-world-config.xml

# All jobs folder
docker cp jenkins:/var/jenkins_home/jobs ./jenkins-jobs-backup
```

### Option 3 — ThinBackup plugin (UI-based)

1. Jenkins → Manage Jenkins → Plugins → Available → search `ThinBackup` → Install
2. Jenkins → Manage Jenkins → ThinBackup → Settings:

| Setting | Example |
|---|---|
| Backup directory | `/var/jenkins_home/thinbackup` |
| Full backup schedule | `0 2 * * 0` (every Sunday 2am) |
| Differential backup schedule | `0 2 * * 1-6` (Mon–Sat 2am) |
| Max backups to keep | `10` |

3. Manual backup: Manage Jenkins → ThinBackup → **Backup Now**
4. Restore: Manage Jenkins → ThinBackup → **Restore**

### Backup comparison

| | Option 1 (Volume) | Option 2 (Config XML) | Option 3 (ThinBackup) |
|---|---|---|---|
| Build history | Yes | No | Yes |
| Easy restore | Yes | Manual | Yes (UI) |
| Scheduled | Manual | Manual | Yes |
| Full DR | Yes | No | Partial |

---

## Jenkins vs GitHub Actions

| | GitHub Actions | Jenkins |
|---|---|---|
| Setup | Minutes | Hours |
| Maintenance | None | You own it |
| Cost | Free tier + paid minutes | Free (but infra costs) |
| Flexibility | Medium | Very high |
| Plugin ecosystem | Growing | Mature (1800+ plugins) |
| Self-hosted | Possible | Native |
| Learning curve | Low | Medium-High |

**When to use GitHub Actions:** Code is on GitHub, small-to-medium teams, open source, zero infra overhead.

**When to use Jenkins:** Enterprise, complex pipelines, on-premise/air-gapped, full infrastructure control.

Many teams use **both**: Jenkins for heavy infra pipelines, GitHub Actions for lightweight PR checks.

---

## Project Structure

```
Jenkins-All/
├── README.md
├── jenkins.ini              # JJB connection config (do not commit credentials)
└── jobs/
    ├── README.md
    ├── hello-world.yaml     # Freestyle job example
    ├── hello-pipeline.yaml  # Pipeline job example
    └── my-folder.yaml       # Folder with jobs inside example
```
