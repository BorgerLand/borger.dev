# Deployment

### Release build

```bash
borger release
```

This produces a `release` folder containing:

- `/release/server` - The final native executable, compiled to run on your OS
- `/release/client` - The static webpage, which can be hosted with any HTTP server capable of serving files

### Prerequisites

Borger is free software, and as such, is unable to provide all the necessary infrastructure and services required for server hosting. You will need:

1. At least one always-on **dedicated server** (VPS, bare metal, compute instance all commonly used). If you're just getting started, [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/) offers a single, small compute instance and will never charge you for it. You may choose to have multiple servers across different global regions. In terms of maximum players that a single server can handle, there isn't yet any concrete data for Borger specifically, but a single powerful server hosting a well-optimized game should theoretically be able to handle more than 1000 CCU (concurrent users). [Most web games](https://webgamedb.com/) never even come close to this.
2. A **domain name** (either pay for one or find a free subdomain), because Safari has some awful [bug](https://www.reddit.com/r/webdev/comments/1pzmhvf/websocket_broken_on_safariios_26/) that prevents it from connecting to a raw IP address. The domain name should point to/fall back to an IPv4 address because a [frightening percentage of the world](https://stats.labs.apnic.net/ipv6) can't connect to IPv6. Some free options:
   - [https://nip.io/](https://nip.io/) - No account required, hardcodes the IP address into the domain. If the IP address changes after you've given out a link, the link is now dead.
   - [https://www.duckdns.org/](https://www.duckdns.org/) - Requires an account, lets you pick out your own name.

3. **TLS certificates** for the domain name, which can be obtained from a free service called [Let's Encrypt](https://letsencrypt.org/), via a tool called [Certbot](https://certbot.eff.org/instructions). Remember that TLS certificates expire. The game server automatically reloads them daily, making use of the fact that Certbot overwrites the same files each time it renews them. Your HTTP server of choice must also be able to handle this.

If enough demand is shown, a paid service will be introduced in order to automate this process. For now, server orchestration and matchmaking are outside the scope of the problems Borger aims to solve.

### Firewall Settings/[Port Forwarding](https://en.wikipedia.org/wiki/Port_forwarding)

| Port | Transport Layer Protocol | Application Protocol | Purpose                       |
| ---- | ------------------------ | -------------------- | ----------------------------- |
| 6969 | UDP                      | WebTransport         | Game Server                   |
| 6996 | TCP                      | WebSocket            | Game Server                   |
| 80   | TCP                      | HTTP                 | Game Client + Certbot Renewal |
| 443  | TCP                      | HTTPS                | Game Client                   |

### Make 'em Move, Hunny

The game server and HTTP server must be running at the same time.

Game server:

```bash
#note there are more flags eg. for using different ports
./server --fullchain /etc/letsencrypt/live/my.domain.com/fullchain.pem --privkey /etc/letsencrypt/live/my.domain.com/privkey.pem
```

HTTP server: There is currently not an included default. You are responsible for choosing one (nginx, Caddy, Apache, plain Node.js script, etc.). It must:

- Serve static files, with the root pointed to the `client` directory. This works well with Certbot's `webroot` plugin.
- It must forward/redirect HTTP to HTTPS in order to enable all required web platform features.
- Use these headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Strict-Transport-Security: max-age=31536000; includeSubDomains
```
