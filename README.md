# VProfile Web Application
####

VProfile is a Java web application packaged as a WAR and deployed to Apache Tomcat. The application uses Spring Framework, Spring Security, Spring Data JPA, JSP, MySQL, Memcached, RabbitMQ, and Elasticsearch.

## Technology Stack

- Java 17 source and target compatibility
- Maven
- Spring Framework 6 and Spring Security 6
- Jakarta Servlet/JSP APIs
- MySQL 8
- Apache Tomcat 10
- Docker and AWS ECR
- GitHub Actions, SonarQube, and Helm-based GitOps deployment

## Repository Layout

```text
src/main/java/       Java application code
src/main/resources/  Application configuration and SQL resources
src/main/webapp/     JSP views, CSS, JavaScript, and static assets
src/test/java/       Unit and integration tests
Docker-files/        Application, database, and Nginx Dockerfiles
.github/workflows/   CI/CD workflow
```

## Prerequisites

- JDK 17 or newer
- Maven 3.9 or newer
- Docker, when building or running the container image
- MySQL, Memcached, RabbitMQ, and Elasticsearch for a complete local runtime

The GitHub Actions workflow uses Temurin JDK 21. Maven compiles the project with Java 17 compatibility settings.

## Build and Test

From the repository root, run:

```bash
mvn clean verify
```

This compiles the application, runs the tests, and creates the WAR at:

```text
target/vprofile-v2.war
```

To generate the Checkstyle report used by CI:

```bash
mvn clean verify checkstyle:checkstyle -B
```

The project also includes SonarQube settings in `sonar-project.properties`. A SonarQube server and authentication token are required to run the scan.

## Configuration

The default settings are in `src/main/resources/application.properties`. By default, the application expects these service endpoints:

| Service | Host | Port | Default credentials/details |
| --- | --- | ---: | --- |
| MySQL | `vprodb` | `3306` | Database `accounts`, user `root`, password `vprodbpass` |
| Memcached | `vprocache01` | `11211` | None |
| RabbitMQ | `vpromq01` | `5672` | `guest` / `guest` |
| Elasticsearch | `localhost` | `9300` | Cluster `vprofile` |

These values are development defaults. Change them before using the application in a shared or production environment. The database image loads `Docker-files/db/db_backup.sql` during initialization.

## Build and Run with Docker

Build the WAR first, then build the Tomcat image:

```bash
mvn clean package -DskipTests
docker build -f Docker-files/app/Dockerfile -t vprofile:local .
docker run --rm -p 8080:8080 vprofile:local
```

Open `http://localhost:8080` after the required backing services are available and reachable using the configured hostnames.

The repository also contains Dockerfiles for the database and Nginx reverse proxy under `Docker-files/db` and `Docker-files/web`. There is currently no Docker Compose file, so those services must be created and networked separately.

## CI/CD Pipeline

The workflow in `.github/workflows/ci.yml` has two paths:

- **Pull requests targeting `main`:** builds the project, runs tests, generates Checkstyle output, runs a SonarQube scan, and checks the SonarQube quality gate.
- **Pushes to `main`:** builds and pushes an image to AWS ECR, then updates `app.image` and `app.tag` in `helm/vprofile/values.yaml` in a separate Helm repository.

Configure these GitHub repository variables:

- `AWS_REGION`
- `ECR_REPOSITORY`
- `HELM_REPO_NAME`
- `SONAR_HOST_URL`

Configure these GitHub repository secrets:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `SONAR_TOKEN`
- `HELM_REPO_USER`
- `GITOPS_PAT`

The deployment image tag is the first seven characters of the commit SHA. The workflow also publishes the `latest` tag.

### Current Dockerfile Note

The CI workflow uses `Docker-files/app/multistage/Dockerfile`. Its build stage clones `https://github.com/hkhcoder/vprofile-project` and checks out its `docker` branch before running Maven. As written, that image is built from the cloned upstream source, not directly from the current checkout. Review this behavior before relying on CI images for changes made only in this repository.

## Application Packaging

The application is deployed as `ROOT.war` to Tomcat 10, which exposes port `8080`. The primary container command is:

```text
catalina.sh run
```

## License

No license file is currently included in this repository.