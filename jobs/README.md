# Jobs

Jenkins job definitions managed with [Jenkins Job Builder](https://jenkins-job-builder.readthedocs.io/).

## Jobs

### hello-world (Freestyle)

A basic freestyle job that prints a hello message and the build number.

- **File:** `hello-world.yaml`
- **Type:** Freestyle
- **Builder:** Shell

### hello-pipeline (Pipeline)

A declarative pipeline job with two stages: Hello and Done.

- **File:** `hello-pipeline.yaml`
- **Type:** Pipeline (inline DSL)
- **Stages:** Hello → Done

### my-folder (Folder + Jobs)

A folder containing both a freestyle and a pipeline job.

- **File:** `my-folder.yaml`
- **Type:** Folder + Freestyle + Pipeline
- **Jobs inside:** `my-folder/hello-world`, `my-folder/hello-pipeline`

---

## Adding a New Job

1. Create a new `.yaml` file in this directory
2. Test it locally:
   ```bash
   jenkins-jobs test jobs/<your-job>.yaml
   ```
3. Push to Jenkins:
   ```bash
   jenkins-jobs --conf jenkins.ini update jobs/<your-job>.yaml
   ```

## Creating Jobs Inside a Folder

Use `folder-name/job-name` as the job name. The folder must be defined before the jobs inside it:

```yaml
- job:
    name: my-folder
    project-type: folder
    description: My folder

- job:
    name: my-folder/my-job
    project-type: freestyle
    builders:
      - shell: echo "Hello!"
```

Nested folders also work: `folder/subfolder/job-name`

---

## Job Types

| `project-type` | Description |
|---|---|
| `freestyle` | Classic freestyle project |
| `pipeline` | Declarative/scripted pipeline (inline DSL) |
| `multibranch` | Multibranch pipeline (reads Jenkinsfile from SCM) |
| `matrix` | Multi-configuration (matrix) project |
| `maven` | Maven project |
| `folder` | Folder to organize jobs |

---

## Notes

- Curly braces `{}` in pipeline DSL must be escaped as `{{}}` in JJB YAML.
- JJB only manages job *configurations* — build history is not tracked here.
- `jenkins-jobs get-jobs` was removed in JJB 6.x. Use `curl.exe` to fetch `config.xml` instead.
- On Windows, always use `curl.exe` not `curl` in PowerShell — `curl` is aliased to `Invoke-WebRequest`.
