# PageMotor Hosting Guide

> **Canonical home:** [documentation.elmspark.com/hosting/vultr/](https://documentation.elmspark.com/hosting/vultr/)
>
> This repository is a mirror. The canonical version with full styling, sidebar navigation, search, and dark mode lives at the link above.

A practical guide to running PageMotor on a VPS. Benchmarks, pricing comparisons, and the exact bootstrap we use to get PageMotor 0.8.2b live on a fresh box in about 30 minutes.

Published by [ElmsPark Studio](https://elmspark.com). Updated April 2026.

---

## Contents

- [Why a VPS, not shared hosting](#why-a-vps-not-shared-hosting)
- [Why Vultr, specifically](#why-vultr-specifically)
- [The $55 test rig pick](#the-55-test-rig-pick)
- [Pick a plan for your size](#pick-a-plan-for-your-size)
- [Which region to pick](#which-region-to-pick)
- [What to tick on the Vultr signup screen](#what-to-tick-on-the-vultr-signup-screen)
- [Let Claude Code do the heavy lifting](#let-claude-code-do-the-heavy-lifting)
- [7 manual steps before Claude Code takes over](#before-claude-code-takes-over-7-manual-steps)
- [The prompt to paste](#the-prompt-copy-paste-edit-three-lines)
- [From empty box to live PageMotor](#from-empty-box-to-live-pagemotor-what-claude-code-does-under-the-hood)
- [Settings people forget](#settings-people-forget-and-later-regret)
- [The cost side](#the-cost-side)
- [Summary](#summary)

---

## Why a VPS, not shared hosting

Shared hosts cap PHP execution time, usually between 30 and 60 seconds, and will not lift it. For a brochure site, that is fine. For PageMotor with AI plugins, ecommerce, or a busy contact form, the cap becomes a ceiling the plugin keeps hitting.

A plugin that waits on an AI API response can easily spend more than a minute on a single request. Shared hosting kills the process halfway through. On a VPS the timeouts are yours to set.

---

## Why Vultr, specifically

In April 2026 we set up a three-way test rig across Vultr (New Jersey), DigitalOcean (New York) and Linode (Newark). Same PageMotor version, same plugins, same scenario on each. Measured disk, CPU, database, API latency, and end-to-end pipeline time.

Vultr won on every single measurement and on price.

| What | Vultr | DigitalOcean | Linode |
|---|---|---|---|
| Sequential disk write | **2,930 MB/s** | 1,223 MB/s | 587 MB/s |
| Random 4K IOPS | **139,264** | 68,894 | 94,856 |
| PHP benchmark | **0.076s** | 0.211s | 0.154s |
| 500 DB inserts | **1,484ms** | 5,445ms | 4,233ms |
| Anthropic API latency | **23.8ms** | 45.5ms | 35.8ms |
| Full pipeline time | **7m 17s** | 10m 38s | 7m 58s |
| Monthly cost | **$55** | $63 | $72 |

Fastest and cheapest at the same time. That almost never happens.

For PageMotor specifically, three of those numbers matter more than the rest. DB inserts matter because the admin is database-heavy. Disk IOPS matter because the page cache, CSS compiler and uploads all hit the disk constantly. API latency matters because AI plugins round-trip to Anthropic dozens of times per session.

---

## The $55 test rig pick

For the test rig we picked **Vultr VX1 Cloud Compute Dedicated, 2 vCPU, 8 GB RAM, 120 GB NVMe, $55/mo, in New Jersey**.

Three reasons:

1. **Dedicated vCPU.** No noisy neighbours. When ten customer sites all hit the box at lunchtime, nobody waits on nobody.
2. **Sized for 100 sites.** Vultr's measured range is 50 to 80 light-traffic PageMotor sites on this tier. We added headroom to cover a USA-wide audience without having to split the fleet later.
3. **New Jersey location.** Covers both US coasts at acceptable latency and had the lowest round-trip to Anthropic's API of the three hosts tested.

VX1 is Vultr's current name for their dedicated-CPU NVMe tier. Older docs may call it "Cloud Compute Dedicated". Same product.

---

## Pick a plan for your size

Vultr charges the same USD price in every data centre, so location does not change the bill. Only the plan does. Three profiles below, with pricing verified on vultr.com/pricing/ in April 2026.

### Solo owner with 1 or 2 PageMotor sites

**Cloud Compute, High Performance, 1 vCPU, 2 GB RAM, 50 GB NVMe — $12/mo.**

A genuinely capable box for a couple of brochure sites, a contact form, EP Email, EP Newsletter, and a Stripe-backed ecommerce setup. AI plugins will run, though heavy generations may hit the 2 GB RAM ceiling. For that workload, step up to High Performance 2 vCPU / 4 GB / 100 GB NVMe at $24/mo.

Auto Backups add $2.40/mo.

### SME with one main site and real traffic

**Cloud Compute, High Performance, 2 vCPU, 4 GB RAM, 100 GB NVMe — $24/mo.**

The sweet spot for a business running one serious site. Consultancy, small ecommerce shop, service business with booking and contact forms running all day. 100 GB NVMe is enough headroom for years of images, invoices and database growth. AI plugins run without drama. If sustained daily traffic regularly exceeds a few thousand visitors, step up to 4 vCPU / 8 GB / 180 GB NVMe at $48/mo.

Auto Backups add $4.80/mo.

### Service provider or agency reselling PageMotor hosting

**VX1 Cloud Compute Dedicated, 2 vCPU, 8 GB RAM, 120 GB NVMe — $55/mo.**

Dedicated vCPU is the critical difference. Expect 50 to 100 sites per box depending on traffic. For agencies past about 60 active sites, step up to VX1 4 vCPU / 16 GB / 240 GB NVMe at $111/mo, which roughly doubles the ceiling.

On a shared agency box, enable Auto Backups. $11/mo (20 per cent of the plan) is cheap insurance against one customer's mistake becoming every customer's problem.

### Quick reference

| Profile | Plan | Specs | Price | Capacity |
|---|---|---|---|---|
| Solo, 1-2 sites | High Performance | 1 vCPU / 2 GB / 50 GB NVMe | $12/mo | 1-2 PM sites |
| Solo with AI plugins | High Performance | 2 vCPU / 4 GB / 100 GB NVMe | $24/mo | 2-4 PM sites |
| SME, one busy site | High Performance | 2 vCPU / 4 GB / 100 GB NVMe | $24/mo | 1 busy site + ecommerce |
| SME, heavy traffic | High Performance | 4 vCPU / 8 GB / 180 GB NVMe | $48/mo | 1 busy site + AI plugins |
| Agency, 50-100 sites | VX1 Dedicated | 2 vCPU / 8 GB / 120 GB NVMe | $55/mo | Our test rig pick |
| Agency, 100-200 sites | VX1 Dedicated | 4 vCPU / 16 GB / 240 GB NVMe | $111/mo | Next step up |

Auto Backups add 20 per cent to whichever plan you pick. On production, enable them.

---

## Which region to pick

Same price everywhere. Latency is the only thing that changes. Pick the closest Vultr data centre to your visitors, not yourself.

### UK and Europe

London (UK), Frankfurt (Germany), Paris (France), Amsterdam (Netherlands), Madrid (Spain), Stockholm (Sweden), Warsaw (Poland).

For most UK and Irish users, London is the obvious pick. Frankfurt fits if you have significant German-speaking traffic or want a central-European footprint that reaches Berlin, Vienna and Prague at similar latency.

### USA

New Jersey, Miami, Atlanta, Chicago, Dallas, Seattle, Silicon Valley, Los Angeles.

New Jersey (our test rig choice) covers the whole of the eastern US at low latency and reaches the west coast at around 70 ms, which is fine for browsing. For a heavily west-coast audience, pick Silicon Valley or Los Angeles. Dallas is the sensible middle ground for genuinely coast-to-coast traffic.

### Rest of the world

Toronto, Mexico City, São Paulo, Santiago, Tokyo, Osaka, Seoul, Singapore, Mumbai, Delhi, Bangalore, Sydney, Melbourne, Johannesburg.

Vultr has 32 regions total. Almost always one within 100 ms of your audience.

---

## What to tick on the Vultr signup screen

| Setting | Pick | Why |
|---|---|---|
| Server type | Cloud Compute | Bare Metal, Kubernetes and Managed Databases are wrong shape for a PageMotor host. |
| CPU & Storage | Dedicated CPU, NVMe | Dedicated vCPUs avoid noisy-neighbour latency that PageMotor's admin feels. |
| Location | Closest to visitors | Same price in every region. Pick by audience latency. |
| Image | Ubuntu 24.04 LTS x64 | Security updates until 2029, ships PHP 8.3, all tooling is first-class. |
| Plan | Match to size | $12 solo / $24 SME / $55 agency / $111 scale. |
| Auto Backups | Enable for production | 20 per cent on top of the plan. Skip for a pure test rig only. |
| IPv6 Networking | Enabled (free) | Future-proofs the box at zero cost. |
| SSH Keys | ed25519, upload before first boot | Most important security click on the page. |
| Hostname | Descriptive, eg `pm-prod-01` | Readable in logs and dashboard. |
| Everything else | Default | No startup script, no VPC, no DDoS add-on at this scale. |

Generate an ed25519 key on your Mac with `ssh-keygen -t ed25519` if you do not already have one, and paste the public key (`~/.ssh/id_ed25519.pub`) into Vultr's SSH Keys section before launching.

---

## Let Claude Code do the heavy lifting

If "bash commands" and "edit the Nginx vhost" made your eyes glaze over, you are the reason this section exists. [Claude Code](https://claude.com/claude-code) is a command-line tool from Anthropic that will SSH into your Vultr box, run every command in this guide, and explain each step in plain English as it goes.

You do not need to understand what any of the commands do. You need to understand what you want: PageMotor 0.8.2b running at your domain, with SSL, on a fresh Vultr box. Tell Claude Code that, and it handles the rest.

### What Claude Code does for you

- Runs every `apt install`, config edit, `chmod` and `systemctl` command
- Shows you each command before running it, so you can watch it happen
- Checks the output of each step, stops on errors, asks before retrying
- Reads this guide as the reference, so the box matches what is documented
- Never deletes anything without asking first
- Uses British English and plain language when talking to you

### What Claude Code will NOT do automatically

- Create your Vultr account (you sign up as a human)
- Provision the VPS (you click through Vultr's signup screen)
- Point DNS at the new VPS IP (you edit your domain's DNS)
- Create your admin account (you register at `/admin/` on first visit)
- Install EP plugins (ask Claude Code afterwards as a separate step)

Those five things cross a human-authorisation line: money, identity, DNS ownership, admin rights. The rest, Claude Code does.

---

## Before Claude Code takes over: 7 manual steps

These are the things you do by hand first. Each is a one-time thing. After step 7, paste the prompt in the next section and Claude Code handles everything else.

**1. Buy a domain if you do not already have one.** Any registrar works (Namecheap, Cloudflare, GoDaddy, Porkbun). You need to be able to edit the DNS A record, which every registrar supports.

**2. Install Claude Code on your Mac.** Download from [claude.com/claude-code](https://claude.com/claude-code). Free to install. You pay only for Anthropic API usage, which for this setup costs a few pence at most.

**3. Generate an SSH key on your Mac, one time ever.** Open Terminal and run:

```bash
ssh-keygen -t ed25519
```

Press Enter three times to accept the defaults. Your public key is now at `~/.ssh/id_ed25519.pub`. If you already did this years ago, skip to step 4.

**4. Sign up for Vultr and provision the VPS.** Go to [vultr.com](https://www.vultr.com/), sign up, then provision a VPS using the choices in "What to tick on the Vultr signup screen" above. Paste your SSH public key from step 3 into the SSH Keys field during signup. This is the most important security click on the page.

To copy your public key to the clipboard:

```bash
cat ~/.ssh/id_ed25519.pub | pbcopy
```

Then paste into Vultr's SSH Keys field.

**5. Note the VPS IP address.** Once the VPS finishes provisioning (30 to 60 seconds), Vultr's dashboard shows the public IP. Copy it. You need it for step 6 and for the prompt.

**6. Point your domain's A record at the VPS IP.** Log in to your registrar (or Cloudflare if you use it for DNS). Edit the A record for your root domain to point at the VPS IP from step 5. Propagation takes anywhere from a few minutes to an hour.

Quick check it worked: `dig yoursite.com +short` in Terminal should return the VPS IP.

**7. Download the PageMotor 0.8.2b core files.** Grab the official PageMotor 0.8.2b core ZIP from the forum download area and unzip it somewhere easy to find, like `~/Downloads/pagemotor-0.8.2b/`. Note the path. You need it in the prompt.

The folder should contain `pagemotor.php`, `index.php`, `lib/`, `config-sample.php` and `license.txt`.

---

## The prompt (copy, paste, edit three lines)

Open Claude Code in Terminal, paste the prompt below, edit the three bracketed placeholders to match your setup, then hit Enter and watch it run.

```
I have a fresh Ubuntu 24.04 VPS at [VPS_IP_FROM_VULTR].
I can SSH as root using my ed25519 key at ~/.ssh/id_ed25519.

My domain is [yoursite.com]. The A record already points at the VPS IP.

My PageMotor 0.8.2b core files are in [~/Downloads/pagemotor-0.8.2b/].
The folder contains pagemotor.php, index.php, lib/, config-sample.php, license.txt.

Please follow the "From empty box to live PageMotor" procedure at
https://github.com/ElmsPark-Studio/pagemotor-hosting

Do this end-to-end:
1. Bootstrap the server (nginx, PHP 8.3 FPM with required extensions,
   MariaDB, certbot, fail2ban, UFW allowing only 22/80/443)
2. Configure MariaDB to default to utf8mb4
3. Create the database and a dedicated DB user scoped to it
4. Upload my PageMotor 0.8.2b core to /var/www/mysite on the VPS,
   create user-content subfolders, set ownership to www-data
5. Write config.php with my database credentials and the four
   required constants
6. Create the Nginx vhost with fastcgi_read_timeout 600,
   client_max_body_size 64M, and the /lib/ php denial rule
7. Set PHP max_execution_time to 300 and memory_limit to 256M
8. Run certbot --nginx for SSL
9. Verify the Nginx config is valid and services are running
10. Hit the homepage once with curl to trigger the PageMotor seeder
    (creates tables)
11. Confirm the tables exist in the database

Stop after step 11. Do NOT register an admin account. Do NOT install
EP plugins. I will register via /admin/ in a browser, then ask you to
install plugins as a separate next task.

Rules:
- Tell me what you are about to do before doing it
- Show me the output of each step
- Stop on any error and ask me before continuing
- British English (organise, behaviour, colour)
- No em dashes in any file you write. Use commas or full stops.
- Ask before anything destructive
```

**What to edit:**

1. `[VPS_IP_FROM_VULTR]` with your VPS IP (eg `149.28.47.148`)
2. `[yoursite.com]` with your actual domain (appears twice)
3. `[~/Downloads/pagemotor-0.8.2b/]` with the actual path where you unzipped the core

### After the prompt finishes

Claude Code stops at step 11 and tells you the site is ready to register. Now the manual bit:

1. Open `https://yoursite.com/admin/` in your browser
2. Register your first account. Whoever registers first becomes the administrator.
3. Come back to Claude Code and say:

```
Please install EP Email, EP Newsletter, EP Newsletter SendGrid, EP GDPR,
EP Password Reset and EP Diagnostics on yoursite.com. Then configure
EP Email with SendGrid SMTP using my API key [SG.xxx].
```

**One thing the prompt does not cover yet:** SMTP configuration. Once the plugins are in, you will need a SendGrid API key (free tier is fine) and a verified sender domain. That is another short conversation with Claude Code, not a separate guide.

---

## From empty box to live PageMotor (what Claude Code does under the hood)

If you are using the Claude Code prompt above, you can skip this section. It documents the exact sequence Claude Code runs for you, kept here for transparency and for anyone who prefers to do it by hand.

About 30 minutes start to finish if done manually.

### 1. Bootstrap the server

SSH in as root using your ed25519 key.

```bash
apt update && apt upgrade -y
apt install -y nginx php8.3-fpm php8.3-mysql php8.3-curl php8.3-gd \
  php8.3-mbstring php8.3-xml php8.3-zip php8.3-intl php8.3-bcmath \
  php8.3-opcache mariadb-server certbot python3-certbot-nginx \
  fail2ban ufw unzip

ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

This leaves only ports 22, 80 and 443 open.

### 2. Make MariaDB speak utf8mb4 by default

Without this, emojis land in the database as `?`. Create `/etc/mysql/conf.d/utf8mb4.cnf`:

```ini
[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[client]
default-character-set = utf8mb4
```

Then `systemctl restart mariadb`.

### 3. Create the database

```bash
mysql -u root -e "
CREATE DATABASE mysite_pm CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'mysite_db'@'localhost' IDENTIFIED BY 'your-strong-password';
GRANT ALL ON mysite_pm.* TO 'mysite_db'@'localhost';
FLUSH PRIVILEGES;"
```

Use a dedicated user scoped to this database only. Never put root credentials in `config.php`.

### 4. Drop PageMotor 0.8.2b into place

```bash
mkdir -p /var/www/mysite
cd /var/www/mysite
# Copy pagemotor.php, index.php and lib/ from your local 0.8.2b core
mkdir -p user-content/{themes,plugins,images,uploads,css,tmp}
```

Copy `config-sample.php` to `config.php` and fill in the four database constants. Leave `PM_INSTALL_LOCATION`, `PM_ADMIN_SLUG` and `PM_HTML_CHARSET` blank.

```bash
chown -R www-data:www-data /var/www/mysite
find /var/www/mysite -type d -exec chmod 755 {} \;
find /var/www/mysite -type f -exec chmod 644 {} \;
chmod -R 775 /var/www/mysite/user-content/
```

### 5. The Nginx vhost

Create `/etc/nginx/sites-available/mysite`:

```nginx
server {
    listen 80;
    server_name yoursite.com;
    root /var/www/mysite;
    index index.php index.html;

    fastcgi_read_timeout 600;
    proxy_read_timeout 600;
    send_timeout 600;
    client_max_body_size 64M;

    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options SAMEORIGIN;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_read_timeout 600;
    }
    location ~ ^/lib/.*\.(js|css|woff|woff2|ttf|eot|svg|png|jpg|gif)$ {
        try_files $uri =404;
    }
    location ~ ^/lib/.*\.php$ { deny all; }
    location ~ /\.(ht|git|env) { deny all; }
}
```

Enable it:

```bash
ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

`fastcgi_read_timeout 600` is the critical line. Nginx defaults to 60 seconds, which any AI plugin will blow through on a heavy call.

### 6. PHP settings that match Nginx

Edit `/etc/php/8.3/fpm/php.ini`:

```ini
max_execution_time = 300
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
```

`systemctl reload php8.3-fpm`. PHP timeout shorter than Nginx means surprise 504s. PHP longer than Nginx means processes running past when Nginx gave up. Keep them aligned.

### 7. SSL with Certbot

Point your domain's A record at the Vultr IP, wait for DNS to propagate, then:

```bash
certbot --nginx -d yoursite.com
```

Adds the 443 block, redirects 80 to 443, wires renewal into systemd.

### 8. First page load seeds the database

Visit your domain. PageMotor detects the empty database and runs its seeder automatically. Then visit `/admin/` and register — first account becomes the admin.

### 9. Install essential plugins

Through the admin, install and activate: EP Email, EP Newsletter, EP Newsletter SendGrid, EP GDPR, EP Password Reset. EP Diagnostics is worth having too.

Configure EP Email with SMTP. SendGrid works well — Vultr blocks outbound port 25 so direct SMTP is not an option anyway.

---

## Settings people forget (and later regret)

- **`fastcgi_read_timeout` must be 600 or higher.** Default 60 kills AI plugins mid-call.
- **`max_execution_time = 300`.** Match PHP to Nginx or expect 504s.
- **MariaDB `character-set-server = utf8mb4`.** Without this, emojis corrupt silently.
- **UFW before anything else.** A new server gets scanned within minutes. Close the doors first.
- **ed25519 SSH keys, password auth off.** Once keys are working, set `PasswordAuthentication no` in `/etc/ssh/sshd_config`.
- **opcache on.** Default on Ubuntu 24.04 but verify with `php -m | grep -i opcache`. 3 to 5 times the page speed for free.

---

## The cost side

The $55 Vultr box was sized for 100 PageMotor sites, based on the measured range of 50 to 80 light-traffic sites per box plus headroom for a USA-wide audience. At 100 sites that works out at roughly 45p each per month. Even at the more cautious 50-site figure, under £1 each. Compare to shared hosting at £5 to £15 per site per month, where AI plugins still cannot run reliably.

Important caveat for single-site owners: the $12 High Performance plan is the right answer for one or two sites, not the $55 one. The VX1 plan only pays off past roughly five sites.

---

## Summary

| Question | Answer |
|---|---|
| Which host? | Vultr |
| Which region? | Closest to your visitors (same price everywhere) |
| Which plan? | $12 solo / $24 SME / $55 agency / $111 scale |
| Which OS? | Ubuntu 24.04 LTS |
| Which stack? | Nginx + PHP 8.3 FPM + MariaDB + Certbot |
| Security baseline? | UFW, ed25519 keys, Fail2ban, password auth off |
| Critical timeouts? | PHP 300s, Nginx 600s, matched |
| Time to live? | About 30 minutes from empty box |

---

## Feedback and corrections

Spotted a mistake, an outdated price, or a step that no longer works? Open an issue on this repo. Pricing especially shifts — every figure here was verified in April 2026, but Vultr changes plans and prices regularly, so always cross-check vultr.com/pricing/ before signing up.
