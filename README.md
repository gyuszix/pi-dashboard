# piDash

Self-hosted network monitoring on a Raspberry Pi Zero 2 W. Runs [Uptime Kuma](https://github.com/louislam/uptime-kuma) in Docker, accessible from anywhere over [Tailscale](https://tailscale.com/).

Keeps tabs on everything in the homelab — other Pis, desktops, laptops, Arduinos, websites.

## Screenshots
<img width="2560" height="1051" alt="Screenshot 2026-03-29 at 20 13 43" src="https://github.com/user-attachments/assets/d33843fa-290e-43cb-9415-864e8dfe31ed" />

## Hardware

| Component | Spec |
|-----------|------|
| Board | Raspberry Pi Zero 2 W (512MB RAM) |
| Storage | 364GB microSD |
| Power | 5V/2.5A micro-USB |
| Network | Onboard WiFi (2.4GHz) |

## Software

- **Raspberry Pi OS Lite (64-bit)** — headless
- **Docker** — containerized Kuma
- **Tailscale** — WireGuard mesh VPN
- **Uptime Kuma** — monitoring dashboard

## Setup

### Flash the OS

Flash **Raspberry Pi OS Lite (64-bit)** to a microSD with [balenaEtcher](https://etcher.balena.io/).

Create login credentials on the boot partition:

```bash
openssl passwd -6
echo 'your_user:YOUR_HASH' > /Volumes/bootfs/userconf.txt
```

Boot with a monitor and keyboard connected.

### WiFi

`raspi-config` WiFi (S1) is broken on Bookworm. Use `nmtui` instead:

```bash
sudo nmtui
```

Go to **Activate a connection**, pick your network.

### SSH + hostname

```bash
sudo systemctl enable ssh && sudo systemctl start ssh
sudo hostnamectl set-hostname piDash
sudo nano /etc/hosts   # swap the default hostname for piDash
```

If you're using Ghostty:

```bash
echo 'export TERM=xterm-256color' >> ~/.bashrc
```

### Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Authorize the device from the link. SSH in over the Tailscale IP from now on.

### Swap

512MB isn't enough for Docker + Kuma. Add swap:

```bash
sudo swapoff -a
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Log out and back in.

### Uptime Kuma

```bash
docker run -d \
  --name uptime-kuma \
  --restart=always \
  -p 3001:3001 \
  -v uptime-kuma:/app/data \
  louislam/uptime-kuma:1
```

Open `http://<tailscale-ip>:3001` and create an admin account.

## Monitors

| Device | Type | Target |
|--------|------|--------|
| Raspberry Pi | Ping | Tailscale IP |
| NAS (NFS) | TCP Port | Tailscale IP:2049 |
| SSH host | TCP Port | Tailscale IP:22 |
| Arduino on WiFi | Ping | Local IP |
| Website | HTTP(s) | URL |

Devices not on Tailscale (like an Arduino) get pinged over the local network. Give them a static IP or DHCP reservation.

## Updating Kuma

```bash
docker pull louislam/uptime-kuma:1
docker stop uptime-kuma && docker rm uptime-kuma
docker run -d \
  --name uptime-kuma \
  --restart=always \
  -p 3001:3001 \
  -v uptime-kuma:/app/data \
  louislam/uptime-kuma:1
```

Data lives in the Docker volume, nothing is lost.


## License

MIT
