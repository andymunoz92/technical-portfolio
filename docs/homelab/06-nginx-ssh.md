# 06 — Static web hosting, Nginx, SSH and permissions

[← Index](README.md) · [← Pi-hole](05-pihole.md) · [Next: Troubleshooting →](07-validation-troubleshooting.md)

## What I wanted

A private environment for editing and hosting a simple static HTML/CSS/JavaScript website from a lightweight Debian LXC, without making it public to the internet yet.

## What was deployed

- A **Debian 13 webserver LXC** attached to the Proxmox LAN bridge, with a stable internal IP.
- **Nginx** to serve static files from `/var/www/html`.
- Folders for `css/`, `js/`, and `images/`.
- A non-root **`developer`** account allowed to edit files and log in over SSH with a key.
- A `www-data` group-based file-permission design for content editing.
- Direct HTTP access to a working web page, plus working terminal SSH access.

## Three distinct concepts

1. **Nginx as web server:** listens for HTTP requests, maps URL paths to files, and sends responses such as `200 OK` or `404 Not Found`.
2. **SSH:** authenticates an administrative user and protects the remote terminal session; it is separate from web traffic.
3. **Linux filesystem permissions:** control who may read, write or traverse content directories. Shell access does not automatically grant write access to every directory.

## Read-only checks

```bash
systemctl is-active nginx
sudo nginx -t                         # Validate config syntax (no restart)
ss -lntup | grep ':80'                # Listening socket for HTTP
curl -I http://127.0.0.1/            # Check local HTTP response
ls -ld /var/www/html
id developer
namei -l /var/www/html/index.html
```

## Permissions I configured

The `developer` account was added to `www-data`, the web root's group ownership was set to `www-data`, group read/write permissions were granted where needed, and **setgid on directories** helped new files inherit the directory group.

**Why not `chmod 777`?** That would give everyone write permission. Group-based access is a smaller, clearer authorization scope. In a more mature deployment, web-server processes usually only need read access to deployed static files, and deployment tooling would perform writes.

## SSH key workflow

1. Create or reuse a dedicated SSH key pair on the client.
2. Install the **public** key into the server user's `~/.ssh/authorized_keys`; never put a private key in a project repository.
3. Confirm the shell can connect using an explicit host/IP alias.
4. Configure VS Code Remote-SSH to use the same host, username and identity file.
5. If terminal SSH works but VS Code does not, compare the client application settings, SSH logs, remote agent start-up and extension environment rather than immediately blaming the host service.

A recorded VS Code test returned `No route to host` while direct terminal SSH worked. The specific IDE failure had not been conclusively resolved in the earlier record; **do not claim it is fixed** without retesting.

## HTTP vs HTTPS vs reverse proxy

- **HTTP:** transfers web requests and responses without TLS encryption.
- **HTTPS:** HTTP protected by TLS, commonly TCP 443.
- **Reverse proxy:** receives client requests and passes them to backend services; Nginx can play this role but **was currently used as a static web server**, not a deployed production reverse proxy.
- **DNS:** translates a name to an address; it does not itself publish or host web content.

## Interview statement

> “I created a Debian web-server container with Nginx, used SSH key authentication to manage it, and configured group permissions so a non-root developer could update the site. That taught me how Unix users and groups interact with service accounts. I also learned to separate SSH client problems from server availability, because a terminal connection worked while the VS Code extension had a routing error.”

**Reference:** [Nginx: serving static content](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/).
