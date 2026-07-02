# Docusaurus-v3-Synology
Build and run a Docusaurus v3 site on a Synology NAS using Docker.

Prerequisites:
- SSH access to your NAS as `admin` (or a user with Docker privileges).
- Docker installed on the NAS (or use `sudo` for Docker commands).

Steps:

1) SSH to your NAS

```bash
ssh admin@YOUR_NAS_IP
```

2) Create a working directory and enter it

```bash
mkdir -p /volume1/docker/docusaurus
cd /volume1/docker/docusaurus
```

3) Start a temporary Node container and mount the working folder (this will open a shell)

```bash
sudo docker run -it --rm -v /volume1/docker/docusaurus:/app node:20-bullseye bash
```

4) Inside the container, scaffold a Docusaurus site

```bash
cd /app
npx create-docusaurus@latest mydocs classic
# This creates /app/mydocs with package.json, docusaurus.config.js, src/, static/, etc.
```

5) Exit the container; the new site is now on the NAS at `/volume1/docker/docusaurus/mydocs`

```bash
exit
```

6) Move (or copy) your `Dockerfile` into the `mydocs` folder and switch to that directory.
   Note: ensure the `Dockerfile` is present at `/volume1/docker/docusaurus/Dockerfile` before moving.

```bash
mv /volume1/docker/docusaurus/Dockerfile /volume1/docker/docusaurus/mydocs/
cd /volume1/docker/docusaurus/mydocs
```

7) Build the production image

Use a consistent image tag; this example uses `docusaurus-v3-prod`:

```bash
docker build -t docusaurus-v3-prod .
```

8) Run the site container

```bash
docker run -d \
  --name docusaurus-v3 \
  -p 3000:3000 \
  docusaurus-v3-prod
```

Now open `http://<YOUR_NAS_IP>:3000` in your browser to view the site.

Optional notes:
- If you need a sample `Dockerfile`, ask and I can add a minimal production-ready example.
- To rebuild after editing site files, re-run `docker build` and restart the container.

README updated with a cleaned, corrected workflow.
