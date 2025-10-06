# cyber-bay-sample-app

<div align="center">
  <img src="img/qr-code.png" alt="Scan to access repository" width="200"/>
  <p><em>Scan to access this repository!</em></p>
</div>

---

This is a sample web application showcasing a multi-tier architecture using **Node.js**, **Python (Flask)**, **PostgreSQL**, and **nginx**.

We will walk through and build this app two different ways:
- **Legacy version** with traditional upstream container images.
- **Chainguard version** using minimal, secure-by-default, zero to near-zero CVE container images.

## Architecture

<div align="center">
  <img src="img/architecture_light.png" alt="Multi-tier architecture diagram" width="100%" style="border-radius: 15px;"/>
</div>

---

## Getting Started

### Prerequisites
- Terminal access
- [Docker](https://www.docker.com/) (image runtime)
- [Docker Compose](https://docs.docker.com/compose/) (multi-container build and orchestration)
- [grype](https://github.com/anchore/grype) (for scanning container images)
- Clone this directory and `cd` into it from your terminal: 
```bash
git clone https://github.com/troy-chainguard-dev/cyber-bay-sample-app.git && cd cyber-bay-sample-app
```

---

## 1. Build and Run the Legacy Version

### Container Images Used

<table>
<tr>
<td width="45%">
  <img src="img/legacy-images-simple.png" alt="Legacy container images" width="100%" style="border-radius: 15px;"/>
</td>
<td width="55%" valign="top">
  <br/>
  <h4>🔗 Upstream Docker Images:</h4>
  <ul>
    <li><strong>nginx:</strong> <a href="https://hub.docker.com/_/nginx">docker.io/library/nginx:latest</a></li>
    <li><strong>Node.js:</strong> <a href="https://hub.docker.com/_/node">docker.io/library/node:latest</a></li>
    <li><strong>Python:</strong> <a href="https://hub.docker.com/_/python">docker.io/library/python:latest</a></li>
    <li><strong>PostgreSQL:</strong> <a href="https://hub.docker.com/_/postgres">docker.io/library/postgres:latest</a></li>
  </ul>
</td>
</tr>
</table>

First we will use docker compose to build the app using the legacy images. The following `docker compose` command will recognize the `docker-compose.yaml` file in the root project directory and build our project using public upstream images from Docker Hub for each specific component.  **Note that the --build flag will force Docker to rebuild each container which involves pulling new images.  This may take a long time on a poor network connection!**

```bash
docker compose up -d --build

# Expected output:
[+] Running 8/8
 ✔ backend Built                           0.0s
 ✔ frontend Built                          0.0s
 ✔ nginx Built                             0.0s
 ✔ Network cyber-bay-sample-app_default    0.1s
 ✔ Container legacy-postgres Started       0.3s
 ✔ Container legacy-backend Started        0.3s
 ✔ Container legacy-frontend Started       0.3s
 ✔ Container legacy-nginx Started          0.3s
```

### Verify It’s Running

- To ensure the containers are running we can run the following from a terminal:
```bash
docker ps

# Expected output:
CONTAINER ID   IMAGE                       STATUS         PORTS                    NAMES
9da02e3b2f76   cyber-bay-nginx:latest      Up 3 minutes   0.0.0.0:80->80/tcp       legacy-nginx
26e1462fabb0   cyber-bay-frontend:latest   Up 3 minutes                            legacy-frontend
3cc427943561   cyber-bay-backend:latest    Up 3 minutes   0.0.0.0:5000->5000/tcp   legacy-backend
22f51e9cdff9   postgres:latest             Up 3 minutes   0.0.0.0:5432->5432/tcp   legacy-postgres
```

- Open [http://localhost:80](http://localhost:80) in your browser to view the website. You should see the following:

<div align="center">
  <img src="img/website.png" alt="Course Registration Website" style="border-radius: 15px;"/>
</div>

- Check that the backend API works by running the following from a terminal:

```bash
curl http://localhost:5000
```

You should see the following response: `Hooray! The API works.`

### Scan Legacy Images for CVEs

Now let's scan our running containers for security vulnerabilities. The `grype-scan.sh` script will:
- Detect all running containers from `docker compose`
- Use Grype to scan each container image for known CVEs (Common Vulnerabilities and Exposures)
- Generate a detailed CSV report with vulnerability information

```bash
./scanners/grype-scan.sh
```

The scan results will be saved to `./scanners/scan-results/grype-legacy-images.csv`.

---

## 2. Tear Down the Legacy Stack

To clean everything, including volumes:

```bash
docker compose down -v
```

---

## 3. Build and Run the Chainguard Version

### Container Images Used

<table>
<tr>
<td width="45%">
  <img src="img/chainguard-images-simple.png" alt="Chainguard container images" width="100%" style="border-radius: 15px;"/>
</td>
<td width="55%" valign="top">
  <br/>
  <h4>🐙 Chainguard Images:</h4>
  <ul>
    <li><strong>nginx:</strong> <a href="https://images.chainguard.dev/directory/image/nginx/overview">cgr.dev/chainguard/nginx:latest</a></li>
    <li><strong>Node.js:</strong> <a href="https://images.chainguard.dev/directory/image/node/overview">cgr.dev/chainguard/node:latest</a></li>
    <li><strong>Python:</strong> <a href="https://images.chainguard.dev/directory/image/python/overview">cgr.dev/chainguard/python:latest</a></li>
    <li><strong>PostgreSQL:</strong> <a href="https://images.chainguard.dev/directory/image/postgres/overview">cgr.dev/chainguard/postgres:latest</a></li>
  </ul>
  <p><em>✅ Zero to near-zero CVEs • Minimal attack surface</em></p>
</td>
</tr>
</table>

We will now use Docker Compose to create our Chainguard version of the app by pointing to a specific compose file called `docker-compose-chainguard.yaml`  This compose file will reference the specific `cgr.dev/chainguard/<images>` listed above

```bash
docker compose -f docker-compose-chainguard.yaml up -d --build

# Expected output:
[+] Running 8/8
 ✔ backend Built                           0.0s
 ✔ frontend Built                          0.0s
 ✔ nginx Built                             0.0s
 ✔ Network cyber-bay-sample-app_default    0.1s
 ✔ Container cg-postgres Started           0.3s
 ✔ Container cg-backend Started            0.4s
 ✔ Container cg-frontend Started           0.4s
 ✔ Container cg-nginx Started              0.4s
```

### Verify It's Running

- To ensure the Chainguard-based containers are running we can run the following from a terminal and see all of the **cg** tags on our images and container names:
```bash
docker ps

# Expected output:
CONTAINER ID   IMAGE                                STATUS         PORTS                    NAMES
476abfd23815   cyber-bay-nginx-cg:latest            Up 5 minutes   0.0.0.0:80->80/tcp       cg-nginx
4a12bab4e30b   cyber-bay-frontend-cg:latest         Up 5 minutes                            cg-frontend
5151ef168869   cyber-bay-backend-cg:latest          Up 5 minutes   0.0.0.0:5000->5000/tcp   cg-backend
949fdcf98c9d   cgr.dev/chainguard/postgres:latest   Up 5 minutes   0.0.0.0:5432->5432/tcp   cg-postgres
```

- Open [http://localhost:80](http://localhost:80)

- Check the API:

```bash
curl http://localhost:5000/
```

### Scan Chainguard Images for CVEs

Now let's scan the Chainguard images for security vulnerabilities. The `grype-scan.sh` script will:
- Detect all running containers from `docker compose`
- Use Grype to scan each Chainguard container image for known CVEs
- Generate a detailed CSV report showing the security improvements

```bash
./scanners/grype-scan.sh
```

The scan results will be saved to `./scanners/scan-results/grype-chainguard-images.csv`.

---

## 4. Tear Down the Chainguard Stack

To clean everything, including volumes:

```bash
docker compose down -v
```

---
## Compare Results

After scanning both versions, open the CSV files to review the outputs to compare:

- Total CVEs
- Severity levels (Critical, High, etc.)
- Image size and dependency differences

This highlights the value of using Chainguard's minimal, secure-by-default images like those from Chainguard.

## Extra Credit

~Compare image sizes, SBOMs, and provenance~