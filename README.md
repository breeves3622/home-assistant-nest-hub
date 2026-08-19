# Home Assistant + Google Nest Hub Dashboard Stack

Containerized **Home Assistant Core** repository designed for deployment via **Portainer Stacks** and GitHub ([`breeves3622`](https://github.com/breeves3622)), specifically pre-configured to cast custom Lovelace dashboards to **Google Nest Home Hub** devices.

---

## 📁 Repository Structure

```text
.
├── docker-compose.yml               # Portainer Stack definition (Home Assistant + optional Cloudflare Tunnel)
├── .gitignore                       # Shields secrets, HA database, and runtime logs from GitHub
├── README.md                        # Deployment & Configuration Guide
└── config/                          # Home Assistant configuration volume directory
    ├── configuration.yaml           # Core config: Cast integration, HTTP trusted proxies & CORS headers
    ├── automations.yaml             # Pre-built Nest Hub casting & recast loop automations
    ├── scripts.yaml                 # Custom script definitions
    ├── scenes.yaml                  # Custom scene definitions
    ├── secrets.yaml.example         # Template for sensitive credentials
    └── dashboards/
        └── nest_hub_dashboard.yaml  # Touch-optimized dashboard layout for Google Nest Hub (1024x600)
```

---

## ⚡ Deployment via Portainer

### Step 1: Push Repository to GitHub (`breeves3622`)

Run the following commands in your local terminal:

```bash
git init
git add .
git commit -m "Initial commit: Home Assistant Nest Hub Dashboard Stack"
git branch -M main
git remote add origin https://github.com/breeves3622/<YOUR-REPO-NAME>.git
git push -u origin main
```

*(Note: Sensitive data such as `home-assistant_v2.db`, `.storage/`, and `secrets.yaml` are ignored by `.gitignore` and won't be exposed).*

### Step 2: Deploy Stack in Portainer

1. Open your **Portainer Web Console**.
2. Navigate to **Stacks** -> **Add stack**.
3. Select **Repository** as the build method.
4. Fill in:
   - **Name**: `homeassistant`
   - **Repository URL**: `https://github.com/breeves3622/<YOUR-REPO-NAME>.git`
   - **Repository reference**: `refs/heads/main`
   - **Compose path**: `docker-compose.yml`
5. *(Optional)* Add Environment Variables under **Environment variables**:
   - `TZ`: `America/New_York` (or your timezone)
   - `CLOUDFLARE_TUNNEL_TOKEN`: `<your_cloudflare_tunnel_token>` (if using Cloudflare Tunnel)
6. Click **Deploy the stack**.

---

## 🔒 HTTPS Requirement for Google Nest Hub Casting

> [!IMPORTANT]
> **Why HTTPS is Mandatory**
> Google Nest Hub (Chromecast) runs an embedded web browser that loads `https://cast.home-assistant.io/`.
> Google Chromecast security policy **strictly blocks non-HTTPS (http://) and self-signed certificates**.

### Setting Up HTTPS (Choose One Option):

#### Option A: Cloudflare Tunnel (Recommended & Free)
1. Create a free account at [Cloudflare Dashboard](https://dash.cloudflare.com/) and add a domain.
2. Go to **Zero Trust** -> **Networks** -> **Tunnels** and create a tunnel.
3. Add a Public Hostname rule: `ha.yourdomain.com` pointing to `http://homeassistant:8123` (or your HA container IP).
4. Copy your Tunnel Token and add it as `CLOUDFLARE_TUNNEL_TOKEN` in your Portainer stack environment variables, then enable the `cloudflared` profile in `docker-compose.yml`.

#### Option B: Home Assistant Cloud (Nabu Casa)
1. In Home Assistant UI, go to **Settings** -> **Home Assistant Cloud**.
2. Start a trial or log in. Native HTTPS and 1-click casting work automatically out of the box.

---

## 📺 Pairing & Casting to Google Nest Hub

1. **Complete Initial HA Setup**:
   - Navigate to `http://<PORTAINER_HOST_IP>:8123` and create your admin account.
2. **Add Google Cast Integration**:
   - Go to **Settings** -> **Devices & Services** -> **Add Integration**.
   - Search for **Google Cast** and follow the prompts. Your Google Nest Hub will appear as a `media_player` entity (e.g. `media_player.google_nest_hub`).
3. **Verify CORS & External URL**:
   - In `config/configuration.yaml`, update `external_url` to your public HTTPS domain (`https://ha.yourdomain.com`).
   - Restart Home Assistant in Portainer.
4. **First-Time Cast Authorization**:
   - Open [https://cast.home-assistant.io/](https://cast.home-assistant.io/) in your desktop browser.
   - Click **Connect**, enter your external HA domain, authorize the connection, and cast your `nest-hub-dashboard`.
5. **Automated Recast Loop**:
   - Google Nest Hubs automatically return to ambient mode after 10-15 minutes of inactivity.
   - Open `config/automations.yaml` and verify your `entity_id` matches your Nest Hub (`media_player.google_nest_hub`).
   - The included automation will automatically refresh the cast dashboard during active hours!

---

## 🎨 Customizing your Nest Hub Dashboard

Edit [`config/dashboards/nest_hub_dashboard.yaml`](file:///c:/Users/breev/Documents/antigravity/blissful-hubble/config/dashboards/nest_hub_dashboard.yaml) to customize layout, add sensors, light controls, or custom cards (such as Mushroom Cards).

Changes pushed to GitHub can be re-synced in Portainer by clicking **Update the stack** -> **Pull latest image and re-deploy**.
