# Socket.IO + Apache Reverse Proxy Notes

- [Socket.IO + Apache Reverse Proxy Notes](#socketio--apache-reverse-proxy-notes)
- [1. Introduction](#1-introduction)
- [2. Check Current Transport](#2-check-current-transport)
  - [Server](#server)
  - [Client](#client)
- [3. Force WebSocket Transport](#3-force-websocket-transport)
  - [Server](#server-1)
  - [Client](#client-1)
- [4. Enable Apache Reverse Proxy Modules](#4-enable-apache-reverse-proxy-modules)
- [5. Apache Virtual Host Configuration](#5-apache-virtual-host-configuration)
- [6. Restart Apache](#6-restart-apache)
  - [Standard Apache](#standard-apache)
  - [Bitnami Stack](#bitnami-stack)
- [7. Troubleshooting Tips](#7-troubleshooting-tips)


---

# 1. Introduction

This guide explains how to:

* Check the active Socket.IO transport
* Force WebSocket connections
* Configure Apache as a reverse proxy for Socket.IO
* Enable required Apache modules
* Restart Apache services properly

---

# 2. Check Current Transport

Use this to verify whether the connection is using:

* `polling`
* `websocket`

## Server

```js id="server-transport-check"
io.on('connection', (socket) => {
  console.log(socket.conn.transport.name);
});
```

## Client

```js id="client-transport-check"
console.log(socket.io.engine.transport.name);
```

---

# 3. Force WebSocket Transport

Force WebSocket transport to avoid HTTP long-polling delays.

## Server

```js id="server-force-websocket"
const io = new Server(server, {
  transports: ['websocket']
});
```

## Client

```js id="client-force-websocket"
const socket = io(url, {
  transports: ['websocket']
});
```

---

# 4. Enable Apache Reverse Proxy Modules

Enable the required Apache modules before configuring the reverse proxy.

Run the following commands with `sudo`:

```bash id="enable-apache-modules"
a2enmod proxy
a2enmod proxy_http
a2enmod proxy_wstunnel
a2enmod rewrite
```

---

# 5. Apache Virtual Host Configuration

Example Apache configuration for Socket.IO with WebSocket support.

```xml id="apache-vhost-config"
<VirtualHost *:443>
    ServerName your-domain.com

    SSLEngine on
    ProxyPreserveHost On
    ProxyRequests Off
    RewriteEngine On

    ProxyPass /socket.io/ ws://127.0.0.1:3000/socket.io/
    ProxyPassReverse /socket.io/ ws://127.0.0.1:3000/socket.io/

    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

---

# 6. Restart Apache

## Standard Apache

```bash id="restart-apache"
systemctl restart apache2
```

## Bitnami Stack

```bash id="restart-bitnami"
/opt/bitnami/ctlscript.sh restart
```

---

# 7. Troubleshooting Tips

* Ensure port `3000` is accessible locally.
* Verify SSL certificates are correctly configured.
* Confirm WebSocket upgrades are not blocked by firewalls.
* Check Apache logs:

  * `/var/log/apache2/error.log`
  * `/var/log/apache2/access.log`
* Test WebSocket connections using browser developer tools.
