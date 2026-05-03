+++
title = 'Bash Lab'
date = '2026-05-03T08:37:16-04:00'
weight = 80
draft = false
+++



The Black Hat Bash lab is a Docker-based practice environment from the No Starch Press book
*Black Hat Bash*. It simulates a corporate network with eight machines across two subnets,
giving you a realistic target for the penetration testing techniques covered in the book.

## Prerequisites

Before setting up the lab, verify that your system meets these requirements:

- **Operating system:** Kali Linux (tested on Kali Linux 2023.4)
- **RAM:** 4 GB minimum (setup warns you if your system has less and lets you continue)
- **Disk space:** 40 GB free minimum (setup warns you if space is low and lets you continue)
- **Internet access:** Required to pull Docker images and download tools
- **Permissions:** sudo access required for all setup steps

If Burpsuite is not available, install it before starting:

```bash
sudo apt-get install burpsuite -y
```

## Clone the repository

Clone the Black Hat Bash repository to your local machine:

```bash
git clone https://github.com/dolevf/Black-Hat-Bash.git
cd Black-Hat-Bash
```

## Set up the lab

You can set up the lab two ways: the automated path runs a single command that installs
Docker, deploys all containers, and installs third-party tools. The manual path gives you
control over each step.

### Automated setup (recommended)

Run this command from the repository root:

```bash
sudo make init
```

The script performs these steps in order:

1. Checks prerequisites (OS, RAM, disk space, and internet connectivity).
2. Installs Docker if it is not already present.
3. Deploys all containers via `make deploy`.
4. Installs third-party hacking tools into `~/tools/`.

Progress is logged to `/var/log/lab-install.log`. To watch progress in a second terminal:

```bash
tail -f /var/log/lab-install.log
```

When setup finishes, log out and log back in for shell changes to take effect. The script
adds `rustscan` and `gitjacker` aliases to `~/.bashrc`.

### Manual Docker install

If you prefer to install Docker yourself before running `make deploy`, follow these steps.

1. Add the Docker apt source:

   ```bash
   printf '%s\n' "deb https://download.docker.com/linux/debian bullseye stable" \
     | sudo tee /etc/apt/sources.list.d/docker-ce.list
   ```

2. Import the Docker GPG key:

   ```bash
   curl -fsSL https://download.docker.com/linux/debian/gpg \
     | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-ce-archive-keyring.gpg
   ```

3. Update apt and install Docker:

   ```bash
   sudo apt update -y
   sudo apt install docker-ce docker-ce-cli containerd.io -y
   ```

4. Start the Docker service:

   ```bash
   sudo service docker start
   ```

5. Change into the `lab/` directory and deploy:

   ```bash
   cd lab
   sudo make deploy
   ```

## Verify the lab

After deployment, confirm that all containers are running:

```bash
sudo make status
```

The command prints `Lab is up.` when all eight containers are running.

## Network architecture

The lab creates two Docker bridge networks:

| Network   | Subnet         | Bridge       | Notes                    |
| --------- | -------------- | ------------ | ------------------------ |
| public    | 172.16.10.0/24 | br_public    | Internet-facing machines |
| corporate | 10.1.0.0/24    | br_corporate | Internal machines only   |

The eight lab machines are:

| Machine      | Public IP    | Private IP | Hostname                               | Role                        |
| ------------ | ------------ | ---------- | -------------------------------------- | --------------------------- |
| p-web-01     | 172.16.10.10 | —          | p-web-01.acme-infinity-servers.com     | Web server                  |
| p-ftp-01     | 172.16.10.11 | —          | p-ftp-01.acme-infinity-servers.com     | FTP server                  |
| p-web-02     | 172.16.10.12 | 10.1.0.11  | p-web-02.acme-infinity-servers.com     | WordPress site              |
| p-jumpbox-01 | 172.16.10.13 | 10.1.0.12  | p-jumpbox-01.acme-infinity-servers.com | Pivot point (both networks) |
| c-backup-01  | —            | 10.1.0.13  | c-backup-01.acme-infinity-servers.com  | Backup server               |
| c-redis-01   | —            | 10.1.0.14  | c-redis-01.acme-infinity-servers.com   | Redis server                |
| c-db-01      | —            | 10.1.0.15  | c-db-01.acme-infinity-servers.com      | Database server             |
| c-db-02      | —            | 10.1.0.16  | c-db-02.acme-infinity-servers.com      | WordPress database          |

`p-jumpbox-01` sits on both subnets and serves as the pivot point into the corporate
network. `p-web-02` runs WordPress backed by `c-db-02`.

## Post-provisioning details

After the containers start, the deploy script waits 25 seconds and then runs two
provisioning steps automatically:

1. Adds an iptables rule on `p-web-01` that drops all inbound traffic from the corporate
   subnet (`10.1.0.0/24`).
2. Provisions WordPress on `p-web-02` with these credentials:

| Setting         | Value                            |
| --------------- | -------------------------------- |
| Site title      | ACME Impact Alliance             |
| Admin username  | jtorres                          |
| Admin password  | asfim2ne7asd7                    |
| Admin email     | jtorres@acme-impact-alliance.com |
| Admin login URL | http://172.16.10.12/wp-admin.php |

## Third-party tools

`make init` installs these tools:

| Tool                        | Install method | Notes                                                    |
| --------------------------- | -------------- | -------------------------------------------------------- |
| `whatweb`                   | apt            |                                                          |
| `rustscan`                  | Docker image   | `rustscan/rustscan:2.1.1`; alias added to `~/.bashrc`    |
| `nuclei`                    | apt            |                                                          |
| `linux-exploit-suggester-2` | git clone      | Cloned to `~/tools/linux-exploit-suggester-2`            |
| `gitjacker`                 | install script | Moved to `~/tools/gitjacker`; alias added to `~/.bashrc` |
| `LinEnum.sh`                | wget           | Downloaded to `~/tools/LinEnum.sh`                       |
| `dirsearch`                 | apt            |                                                          |
| `jq`                        | apt            |                                                          |
| `ncat`                      | apt            |                                                          |
| `sshpass`                   | apt            |                                                          |
| `pwncat-cs`                 | pip3           |                                                          |
| `unix-privesc-check`        | apt            | Copied to `~/tools/unix-privesc-check`                   |

The `rustscan` alias runs RustScan through Docker without a direct install:

```bash
rustscan
```

If RustScan fails due to architecture incompatibility on macOS, install it natively:

```bash
brew install rustscan
```

## Manage the lab

Run all management commands from the `lab/` directory with sudo.

### Start the lab

If images are already built, bring the containers back up:

```bash
sudo make deploy
```

### Stop the lab

Shut down all containers and remove volumes:

```bash
sudo make teardown
```

### Rebuild the lab from scratch

Destroy the existing environment and redeploy:

```bash
sudo make rebuild
```

### Destroy the lab completely

Remove all containers, images, and volumes, then prune the Docker system:

```bash
sudo make clean
```