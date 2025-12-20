# Multi-Domain Cloudflare Tunnels with Docker Compose

Manage multiple domains from a single VPS using isolated Cloudflare Tunnels. Perfect for users hosting many domains or services on one server—each tunnel runs independently with its own web-based configuration interface, providing automatic HTTPS, DDoS protection, and zero-trust network access without opening any ports on your VPS.

> **💡 Important Note:** This multi-tunnel setup is designed for users managing domains across **different Cloudflare accounts**. If all your domains are managed under a **single Cloudflare account**, you only need **one tunnel**—a single Cloudflare Tunnel can handle alot of domains and services within the same account. Use multiple tunnels when you need complete isolation between different Cloudflare accounts or for organizational separation (e.g., production, staging, development environments).

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

## 📑 Table of Contents

- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Quick Start & Installation](#-quick-start--installation)
  - [Step 1: Clone the Repository](#step-1-clone-the-repository)
  - [Step 2: Launch the Containers with Docker Compose](#step-2-launch-the-containers-with-docker-compose)
  - [Step 3: Configure Each Tunnel via the Web UI](#step-3-configure-each-tunnel-via-the-web-ui)
- [Security & Firewall Configuration](#-security--firewall-configuration)
- [Contributing](#-contributing)

## 🌟 Features

- **Isolated Tunnels:** Run completely separate Cloudflare tunnels, each with its own configuration and credentials.
- **Web UI Management:** Easily manage each tunnel via a secure web UI provided by the wisdomsky/cloudflared-web Docker image.
- **Persistent Configuration:** Tunnel configurations are stored in local volumes, ensuring they survive container restarts or updates.
- **Simplified Deployment:** Use a single docker-compose.yml file to spin up your entire tunnel infrastructure.
- **Secure by Design:** Leverages Cloudflare's secure tunneling to expose local services without opening public ports on your VPS for those services.

## 📋 Prerequisites

Before you begin, ensure you have the following:

- **A VPS:** A server running a modern Linux distribution (like Ubuntu 22.04+).
- **Docker & Docker Compose:** Installed and running on your VPS.
  - [Install Docker](https://docs.docker.com/engine/install/)
  - [Install Docker Compose](https://docs.docker.com/compose/install/)
- **A Cloudflare Account:** With at least one domain added and managed by Cloudflare.

## 🚀 Quick Start & Installation

Follow these steps to get your multi-tunnel setup running in minutes.

### Step 1: Clone the Repository

First, clone this repository to your VPS

```bash
git clone https://github.com/MTDASD/cloudflare-tunnel-multi-domain-docker-single-vps.git
cd cloudflare-tunnel-multi-domain-docker-single-vps
```

### Step 2: Launch the Containers with Docker Compose

> **📌 Note:** The `docker-compose.yml` file uses `/mnt/storage/` as the base path for volume storage. This path may not exist on all systems. You can either:
> - Create the directory: `sudo mkdir -p /mnt/storage/cloudflared{,2,3}/config`
> - Modify the volume paths in `docker-compose.yml` to use a location that exists on your system (e.g., `./cloudflared/config`)

After configuring your `docker-compose.yml` file with your specific tunnel settings and service mappings, launch the containers in detached mode (`-d`).

```bash
docker compose up -d
```

### Step 3: Configure Each Tunnel via the Web UI

Each tunnel's management interface is now accessible on a different port on your VPS.

- **Tunnel 1:** `http://<your-vps-ip>:1111`
- **Tunnel 2:** `http://<your-vps-ip>:2222`
- **Tunnel 3:** `http://<your-vps-ip>:3333`

> **⚠️ Important:** Before accessing these URLs, please read the **Security & Firewall Configuration** section below to protect these management interfaces.

For each tunnel, follow these steps:

1. **Access Cloudflare Zero Trust Dashboard**
   - Navigate to [Cloudflare Zero Trust](https://one.dash.cloudflare.com/)
   - Go to **Networks** → **Tunnels**
   - Click **Create a tunnel**

2. **Create a New Tunnel**
   - Select **Cloudflared** as the connector type
   - Give your tunnel a descriptive name (e.g., `vps-tunnel-1`, `vps-tunnel-2`, `vps-tunnel-3`)
   - Click **Save tunnel**

3. **Copy the Tunnel Token**
   - After creating the tunnel, Cloudflare will display a token (a long string starting with `ey...`)
   - Copy this token to your clipboard

4. **Configure the Tunnel in Web UI**
   - Open the web UI for the corresponding tunnel in your browser:
     - For `cloudflared`: `http://<your-vps-ip>:1111`
     - For `cloudflared2`: `http://<your-vps-ip>:2222`
     - For `cloudflared3`: `http://<your-vps-ip>:3333`
   - Paste the token you copied from Cloudflare Zero Trust into the token field
   - Click **Save** to store the configuration
   - Click **Start** to activate the tunnel

5. **Verify Tunnel Status**
   - Return to the Cloudflare Zero Trust dashboard
   - Navigate to **Networks** → **Tunnels**
   - Your tunnel should now appear as **Healthy** with a green status indicator
   - You can now configure routes and public hostnames for this tunnel

6. **Repeat for All Tunnels**
   - Repeat steps 1-5 for each additional tunnel (`cloudflared2`, `cloudflared3`, etc.)
   - Each tunnel operates independently with its own token and configuration

## 🔒 Security & Firewall Configuration

**Critical:** The web UI ports (1111, 2222, 3333) provide administrative access to your Cloudflare tunnels. Unauthorized access could allow attackers to reconfigure your tunnels, modify tokens, or redirect traffic. It is essential to restrict access to these ports immediately after initial setup.

### Recommended Security Measures:

#### Option 1: Firewall Rules (Recommended)

Use `ufw` (Uncomplicated Firewall) to restrict access to the Web UI ports to your IP address only:

```bash
# Allow access only from your IP address
sudo ufw allow from YOUR_IP_ADDRESS to any port 1111
sudo ufw allow from YOUR_IP_ADDRESS to any port 2222
sudo ufw allow from YOUR_IP_ADDRESS to any port 3333

# Enable the firewall if not already enabled
sudo ufw enable
```

Replace `YOUR_IP_ADDRESS` with your actual public IP address.


#### Option 2: SSH Tunnel

Access the web UI through an SSH tunnel for enhanced security.

#### Option 3: VPN Access Only

Configure the web UI to be accessible only through a VPN connection.

#### Option 4: Close Ports After Configuration

After configuring all tunnels, you can block external access to the web UI ports entirely:

```bash
sudo ufw deny 1111
sudo ufw deny 2222
sudo ufw deny 3333
```

You can re-enable access temporarily when needed for reconfiguration.

## 📚 Usage Examples

After setting up your tunnels, you need to configure routes in Cloudflare Zero Trust to expose your local services through secure HTTPS domains.

### Example: Exposing a Local Web Service

Let's say you have a web application running locally on port 8080 (e.g., a Node.js app, Python Flask server, or any HTTP service).

1. **Access Your Tunnel Configuration**
   - Go to [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com/)
   - Navigate to **Networks** → **Tunnels**
   - Click on your tunnel (e.g., `vps-tunnel-1`)

2. **Add a Public Hostname**
   - Click the **Public Hostname** tab
   - Click **Add a public hostname**

3. **Configure the Route**
   - **Subdomain:** Enter your subdomain (e.g., `app` or `api`)
   - **Domain:** Select your domain from the dropdown (e.g., `example.com`)
   - **Path:** Leave blank (or specify a path like `/api` if needed)
   - **Service:**
     - **Type:** Select `HTTP`
     - **URL:** Enter `localhost:8080` (or your service's address)

4. **Save and Test**
   - Click **Save hostname**
   - Your local service on `http://localhost:8080` is now accessible via `https://app.example.com`
   - Cloudflare automatically provides SSL/TLS encryption (HTTPS on port 443)

### Common Use Cases

| Local Service | Port | Cloudflare Domain | Description |
|--------------|------|-------------------|-------------|
| Web App | `localhost:8080` | `https://app.example.com` | Main application |
| API Server | `localhost:3000` | `https://api.example.com` | REST API |
| Admin Panel | `localhost:9000` | `https://admin.example.com` | Admin dashboard |
| Database UI | `localhost:5432` | `https://db.example.com` | Database management |
| Monitoring | `localhost:3001` | `https://monitor.example.com` | System monitoring |

### Benefits of This Setup

- 🔒 **Automatic HTTPS:** Cloudflare provides free SSL/TLS certificates
- 🌐 **No Port Forwarding:** Your VPS firewall stays closed - only the Cloudflare tunnel needs outbound access
- 🛡️ **DDoS Protection:** Cloudflare's network protects your services
- 🚀 **CDN & Caching:** Benefit from Cloudflare's global CDN
- 🔐 **Access Control:** Use Cloudflare Access to add authentication to any service

### Multiple Services on Different Tunnels

You can route different domains through different tunnels for isolation:

- **Tunnel 1 (`cloudflared`):** `https://public-app.example.com` → Production services
- **Tunnel 2 (`cloudflared2`):** `https://staging.example.com` → Staging environment
- **Tunnel 3 (`cloudflared3`):** `https://dev.example.com` → Development environment

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Made with ❤️ for the Cloudflare and Docker community** 
