# Cloud Computing – Lab 05: Persistency and Object Storage
**Master in Computer Engineering – Mobile Computing (2025/2026)**

**Date:** 24th March 2026

## Worksheet Briefing
The provided application now supports user registration and profile picture uploads. 
The code relies on a relational database (PostgreSQL) and an AWS S3-compatible Object Storage service (MinIO). This worksheet's goals are: orchestrate new stateful services; ensure their data survives container restarts; and route them securely through the Nginx Proxy Manager Gateway.

---

## Phase 1: Architecture Review
Explore the repository source code, that includes:
1. `/frontend`: An updated web dashboard with an image upload form.
2. `/api`: An updated Node.js API. 
   * *Check the `server.js` file to see which environment variables the developers expect you to inject for Postgres and S3!*

**Architectural Requirements:**
* **Zero-Trust Network:** No backend services (API, Postgres, MinIO) may expose ports to the host.
* **State Persistence:** Both the database and the object storage must use Docker Volumes so data is not lost when containers restart.
* **Edge Routing:** Use `*.meicm.pt` (which resolves to `127.0.0.1`) to access the Gateway.

---

## Phase 2: The Infrastructure Challenge
Update your `docker-compose.yml` to include the following **five** services:

1. **`proxy-manager`**: Your API Gateway (Port 80 and 81 exposed to host).
2. **`frontend-web`**: Built from the `/frontend` directory.
3. **`api-service`**: Built from the `/api` directory.
   * *Hint: You must define `DB_HOST`, `DB_USER`, `DB_PASS`, `S3_ENDPOINT`, `S3_ACCESS_KEY`, and `S3_SECRET_KEY` as environment variables here.*
4. **`postgres-db`**: 
   * Use the `postgres:15-alpine` image.
   * Set the required Postgres environment variables for user, password, and database name.
   * Mount a volume to `/var/lib/postgresql/data`.
5. **`minio-storage`**:
   * Use the `minio/minio:RELEASE.2025-01-20T14-49-07Z` image.
   * Set `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD`.
   * Start command: `server /data --console-address ":9001"`
   * Mount a volume to `/data`.

Boot the architecture in detached mode:
`docker compose up -d --build`

---

## Phase 3: The Edge Routing Challenge
You need to configure your API Gateway to route traffic to your new services. 

1. Access the Nginx Proxy Manager UI (`http://localhost:81`).
2. Route `app.meicm.pt` to your Frontend container.
3. Route `api.meicm.pt` to your API container.
4. Route `storage.meicm.pt` to your MinIO container's Web UI (Port `9001`).
5. Route `s3.meicm.pt` to your MinIO container's API port (Port `9000`).

---

## Phase 4: S3 Bucket Initialization
Before the API can upload images, the S3 bucket must exist!
1. Navigate to `http://storage.meicm.pt` and log in with your MinIO root credentials.
2. Create a new bucket called `profile-pictures`.
3. Set the bucket's Access Policy to **Public** (so the frontend can display the images).

---

## Phase 5: Verification & Disaster Recovery Test
1. Go to `http://app.meicm.pt`. Register a user and upload a profile picture.
2. Verify the picture appears on the dashboard and in the MinIO Web UI.
3. **The Chaos Monkey Test:** In your terminal, run `docker compose down` to destroy your entire infrastructure. 
4. Run `docker compose up -d` to bring it back. 
5. Refresh your browser. If your Docker Volumes were configured correctly, your user data and profile picture will still be there!