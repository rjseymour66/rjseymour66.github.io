+++
title = 'Configuration'
date = '2025-08-23T17:39:02-04:00'
weight = 10
draft = false
+++

## systemd configuration

You should start and stop your servers with your operating system's initialization daemon, such as Ubuntu's systemd. Do not run your application as a daemon---use an initialization daemon to manage the execution of the application, including rules for service health and restarts.

Here is an example unit file for systemd:

```ini
[Unit]
Description=My Test Service         # Human-readable name for service
After=network.target                # Start only after system networking is up

[Service]
ExecStart=/usr/local/bin/myapp      # Command to start service

Restart=on-failure                  # systemd restarts service on failure
RestartSec=5

Environment=PORT=4002               # Environment variable
Environment=APP_ENV=production

User=myuser                         # Linux user acct that runs the app (instead of root)
Group=myuser                        # Linux group acct that runs the app

WorkingDirectory=/home/myuser/app   # Process cwd that might store config files or write logs to

StandardOutput=journal              # Set stdout to system journal
StandardError=journal               # Set stderr to system journal

[Install]
WantedBy=multi-user.target          # Start service when system is in multi-user mode (normal startup)
```

These commands manage the service with `systemctl`

```bash
systemctl reload myapp  # activate new configs without bringing service down
systemctl enable myapp  # start service when server boots
systemctl start myapp   # start the unit
systemctl stop myapp    # stop the unit
systemctl status myapp  # check unit status
```

### Creating a systemd service

Configure a Linux server to manage an application with systemd:

1. Copy your Go binary to a directory in your `$PATH`:
   ```bash
   cp app /usr/local/bin/
   chmod +x /usr/local/bin/app
   ```
1. Create a user for the service. Creating a dedicated user for a service ensures that the user can access only the files you grant to them and isolates the service from others in case it is compromised.
   
   This account creates a system account (`-r`) named `serviceuser` without shell access (`-s /bin/false`):
   ```bash
   useradd -r -s /bin/false serviceuser
   ```
2. Create a working directory for the app. This is where you can store configuration files or write logs, and it ensures filesystem saftey:
   ```bash
   mkdir -p /home/username/app
   chown -R username:username /home/username/app
   ```
3. Create a unit file for your service and store it in `/etc/systemd/system/app.service`:
   ```bash
   vim /etc/systemd/system/app.service
   # add file contents
   ```
4. Run the following `systemctl` commands so systemd loads the new service unit file, manages the service at boot, and starts the service:
   ```bash
   systemctl daemon-reload  # load new unit file
   systemctl enable app     # enable app at boot
   systemctl start app      # start service
   systemctl status app     # verify that the service started
   ```
5. Lastly, verify that the service is logging messages to the journal:
   ```bash
   journalctl -u app -f
   ```