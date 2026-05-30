# Trend Intelligence System

A self-hosted workflow automation platform for building trend intelligence pipelines using [n8n](https://n8n.io/). This project runs n8n in Docker so you can connect APIs, schedule jobs, and automate data collection without writing a full backend from scratch.

---

## What This Project Does

| Component | Purpose |
|-----------|---------|
| **n8n** | Visual workflow builder for automating tasks (API calls, data transforms, alerts) |
| **Docker Compose** | Runs n8n in a container with persistent storage |
| **`n8n_data/`** | Local volume for workflows, credentials, and execution history (not committed to Git) |

Use this repo to spin up n8n locally, build trend-monitoring workflows in the UI, and share the infrastructure setup with others via GitHub.

---

## Prerequisites

Before you start, install:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (includes Docker Compose)
- [Git](https://git-scm.com/downloads)
- A GitHub account

**Minimum resources:** ~512 MB free disk space for n8n data.

---

## Quick Start

### 1. Clone the repository

```bash
git clone git@github.com:nirjalatripathi/Trend-intelligence-system.git
cd Trend-intelligence-system
```

If you use HTTPS instead of SSH:

```bash
git clone https://github.com/nirjalatripathi/Trend-intelligence-system.git
cd Trend-intelligence-system
```

### 2. Start n8n

```bash
docker compose up -d
```

On older Docker installs, use:

```bash
docker-compose up -d
```

### 3. Open the n8n dashboard

| Setting | Value |
|---------|-------|
| URL | http://localhost:5678 |
| Username | `admin` |
| Password | `admin123` |

### 4. Stop n8n

```bash
docker compose down
```

### 5. View logs

```bash
docker compose logs -f n8n
```

---

## Project Structure

```
trend-intelligence-system/
├── docker-compose.yml    # n8n service definition
├── .gitignore            # Ignores runtime data and secrets
├── README.md             # This file
└── n8n_data/             # Created at runtime (gitignored)
    ├── database.sqlite   # n8n local database
    └── ...               # workflows, credentials, logs
```

### What gets committed vs ignored

| Tracked in Git | Ignored (local only) |
|----------------|----------------------|
| `docker-compose.yml` | `n8n_data/` |
| `README.md` | `.env` |
| `.gitignore` | |

**Important:** Workflows and API keys you create inside n8n are stored in `n8n_data/`. Export important workflows from the n8n UI if you want them in version control.

---

## Configuration

Environment variables are set in `docker-compose.yml`:

| Variable | Default | Description |
|----------|---------|-------------|
| `TZ` | `Asia/Kathmandu` | Timezone for schedules and logs |
| `N8N_BASIC_AUTH_ACTIVE` | `true` | Enables login protection |
| `N8N_BASIC_AUTH_USER` | `admin` | Web UI username |
| `N8N_BASIC_AUTH_PASSWORD` | `admin123` | Web UI password |

### Using a `.env` file (recommended)

Create a `.env` file in the project root (already gitignored):

```env
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_strong_password_here
TZ=Asia/Kathmandu
```

Then reference it in `docker-compose.yml`:

```yaml
environment:
  - TZ=${TZ}
  - N8N_BASIC_AUTH_ACTIVE=true
  - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
  - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
```

Change the default password before exposing this to a network.

---

## Push This Project to GitHub

Your repo is already linked to:

**https://github.com/nirjalatripathi/Trend-intelligence-system**

### First-time setup (new machine or new fork)

1. **Create a repo on GitHub** (if you do not have one yet)
   - Go to [github.com/new](https://github.com/new)
   - Name it `Trend-intelligence-system` (or any name you prefer)
   - Do **not** initialize with a README if you already have local code

2. **Configure Git identity** (one time per machine)

   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

3. **Add the remote** (only if `git remote -v` shows nothing)

   SSH:

   ```bash
   git remote add origin git@github.com:YOUR_USERNAME/Trend-intelligence-system.git
   ```

   HTTPS:

   ```bash
   git remote add origin https://github.com/YOUR_USERNAME/Trend-intelligence-system.git
   ```

4. **Stage, commit, and push**

   ```bash
   git add .
   git commit -m "Add project documentation and n8n setup"
   git push -u origin main
   ```

### Push updates after you change files

```bash
git status
git add .
git commit -m "Describe what you changed"
git push
```

### SSH vs HTTPS

| Method | When to use |
|--------|-------------|
| **SSH** (`git@github.com:...`) | Recommended if you have [SSH keys set up](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) |
| **HTTPS** (`https://github.com/...`) | Easier on new machines; GitHub will prompt for credentials or a [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) |

Switch remote URL:

```bash
git remote set-url origin https://github.com/YOUR_USERNAME/Trend-intelligence-system.git
```

---

## Working with n8n Workflows

### Export a workflow (backup or share)

1. Open n8n at http://localhost:5678
2. Open a workflow → **⋯** menu → **Download**
3. Save the `.json` file (optionally commit it in a `workflows/` folder)

### Import a workflow

1. **Workflows** → **Import from File**
2. Select a `.json` export

### Backup all n8n data

```bash
# Windows (PowerShell)
Copy-Item -Recurse .\n8n_data .\n8n_data_backup_$(Get-Date -Format yyyyMMdd)

# macOS / Linux
cp -r ./n8n_data ./n8n_data_backup_$(date +%Y%m%d)
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Port `5678` already in use | Stop the other service or change the port in `docker-compose.yml` to `"6789:5678"` |
| Container will not start | Run `docker compose logs n8n` and check permissions on `n8n_data/` |
| Forgot login password | Update `N8N_BASIC_AUTH_PASSWORD` in `docker-compose.yml` or `.env`, then `docker compose up -d` |
| Workflows missing after restart | Ensure `./n8n_data` is mounted; do not delete that folder |
| `git push` rejected | Run `git pull --rebase origin main`, resolve conflicts, then push again |

---

## Security Notes

- Default credentials (`admin` / `admin123`) are for **local development only**.
- Do not commit `.env` or `n8n_data/` — they are listed in `.gitignore`.
- For production: use HTTPS behind a reverse proxy, a strong password, and consider PostgreSQL instead of SQLite.

---

## Resources

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Docker image](https://hub.docker.com/r/n8nio/n8n)
- [Docker Compose docs](https://docs.docker.com/compose/)
- [GitHub: Push code to a remote](https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository)

---

## License

Add a license file (e.g. `LICENSE`) if you plan to open-source this project publicly.
