# Docusaurus-v3-Synology
Build and run a Docusaurus v3 site on a Synology NAS using Docker.

This project supports two setup options for release 2.0.

## Option A: Synology Container Manager setup

1. Download `my-website.zip` from this GitHub repository.
2. Create a folder named `docusaurus` inside `/volume1/docker`.
3. Place the files from `my-website.zip` into the newly created `docusaurus` folder.
4. Open Container Manager and add the image from Docker Hub: https://hub.docker.com/r/smartbrotha/docusaurus-v3-synology/
5. Create a new project.
6. Set the path and select the folder from step 2.
7a. Select Source, then upload the Docker YAML file from this GitHub repository.
7b. Add the contents of the YAML file.
8. Check the box to start the project once it is created.
9. Your site will be available at http://localhost:3000/.
10. Use your favorite Markdown editor to open your Docusaurus folder and make changes.
11. Once changes are done, you can restart the container and it will automatically run `npm run build` and then `npm run serve` to display the changes.

## Option B: Current Docker workflow

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
npx create-docusaurus@latest my-website classic
# This creates /app/my-website with package.json, docusaurus.config.js, src/, static/, etc.
```

5) Exit the container; the new site is now on the NAS at `/volume1/docker/docusaurus/my-website`

```bash
exit
```

6) Move (or copy) your `Dockerfile` into the `my-website` folder and switch to that directory.
   Note: ensure the `Dockerfile` is present at `/volume1/docker/docusaurus/Dockerfile` before moving.

```bash
mv /volume1/docker/docusaurus/Dockerfile /volume1/docker/docusaurus/my-website/
cd /volume1/docker/docusaurus/my-website
```

7) Build the production image

Use a consistent image tag; this example uses `docusaurus-v3-syno`:

```bash
docker build -t docusaurus-v3-syno .
```

8) Run the site container

```bash
docker run -d \
  --name docusaurus \
  --init \
  --restart always \
  -p 3000:3000 \
  -p 35729:35729 \
  -v /volume1/docker/docusaurus/my-website/blog:/app/docusaurus/blog \
  -v /volume1/docker/docusaurus/my-website/docs:/app/docusaurus/docs \
  -v /volume1/docker/docusaurus/my-website/src:/app/docusaurus/src \
  -v /volume1/docker/docusaurus/my-website/static:/app/docusaurus/static \
  -v /volume1/docker/docusaurus/my-website/README.md:/app/docusaurus/README.md \
  -v /volume1/docker/docusaurus/my-website/sidebars.js:/app/docusaurus/sidebars.js \
  -w /app/docusaurus \
  docusaurus-v3-syno \
  /bin/bash -c "npm run build && npm run serve -- --host 0.0.0.0 --no-open"
```

Now open `http://<YOUR_NAS_IP>:3000` in your browser to view the site.

Files will need to be edited using the Docker container terminal via Container Manager.

Optional notes:
- If you need a sample `Dockerfile`, ask and I can add a minimal production-ready example.
- To rebuild after editing site files, re-run `npm run build` and then `npm run serve`.
