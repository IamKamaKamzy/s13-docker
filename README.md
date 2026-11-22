# S13 Training Camps

A self-contained student training website that runs as one Docker container.

## Single-container architecture

The `app` container provides everything:

- Responsive HTML, CSS, and JavaScript website
- Course catalog API
- Student registration and login API
- Enrollment API
- File-based demo persistence through the `app-data` Docker volume

The application uses only the Node.js standard library and has no package dependencies. Every commit pushed to a branch triggers a GitHub Actions Docker build, publishes the image to Docker Hub as `iamkamakamzy/automation` with commit-SHA and `latest` tags, and starts a temporary container to check its health and courses APIs. Pull-request updates build the image without publishing or starting a container.

## Tools used

- HTML5, CSS3, and vanilla JavaScript
- Node.js 22
- Docker and Docker Compose
- GitHub Actions
- Git and curl

## Run

```bash
docker build -t s13-training-camps:latest .
docker volume create s13-training-camps-data
docker run -d --name s13-training-camps --restart unless-stopped -p 8080:8080 -v s13-training-camps-data:/data s13-training-camps:latest
```

Open <http://localhost:8080>.

## Verify

```bash
docker ps --filter name=s13-training-camps
curl http://localhost:8080/health
curl http://localhost:8080/api/courses
```

## Stop

```bash
docker rm -f s13-training-camps
```

Use `docker volume rm s13-training-camps-data` only when you intentionally want to delete saved demo users and enrollments.

## CI/CD

`.github/workflows/docker-build.yml` builds the image on GitHub's hosted runner for every push and pull-request update, and publishes it to Docker Hub for branch pushes. For pushes, CI also runs a temporary container smoke test and shows its status, ports, and logs in the Actions run summary. The container stops with the job; Docker Hub stores the image but does not host running containers. Add a Docker Hub access token that can push to `iamkamakamzy/automation` as the repository Actions secret `DOCKERHUB_TOKEN`. The published image uses both the full commit SHA and `latest` tags.
