---
title: "Keycloak Ssl Adventure"
date: 2025-07-03T19:54:53-05:00
draft: true
---
# The Great Keycloak SSL Adventure: A Tale of Mixed Content, Proxy Headers, and Lessons Learned

*A developer's journey from CloudFront complexity to simple SSL success*

## The Problem That Started It All

Picture this: you've got Keycloak running smoothly behind AWS CloudFront and Application Load Balancer (ALB). Everything seems perfect until you open your browser's developer console and see the dreaded red errors:

```
Blocked loading mixed active content "http://keycloack.example.com/resources/master/admin/en"
Content-Security-Policy: The page's settings blocked the loading of a resource...
```

Mixed content errors. The bane of every developer trying to serve a secure application. Your users access the site via HTTPS, but Keycloak is generating HTTP URLs for its resources, causing browsers to block them for security reasons.

## First Attempt: The "Simple" Fix That Wasn't

Like any reasonable developer, I started with what seemed like the obvious solution: configure Keycloak to understand it's behind an HTTPS proxy. How hard could it be?

**Spoiler alert: Very hard.**

The initial advice I gave was textbook perfect:
- Set `KC_PROXY=edge` to enable proxy mode
- Configure CloudFront to send `X-Forwarded-Proto: https` headers
- Add hostname configuration to Keycloak

But after implementing these changes, the mixed content errors persisted. Keycloak was still generating HTTP URLs.

```javascript
// What we kept seeing in the browser:
{
  "serverBaseUrl": "http://keycloack.example.com",  // ❌ Still HTTP!
  "adminBaseUrl": "http://keycloack.example.com",
  "authUrl": "http://keycloack.example.com"
}
```

## Down the Rabbit Hole: AWS Proxy Chain Debugging

This is where things got interesting (and frustrating). We had a complex chain:

```
Browser (HTTPS) → CloudFront (HTTPS) → ALB (HTTP) → Keycloak (HTTP:8080)
```

Each component needed to properly forward proxy information. The debugging journey involved:

### CloudFront Configuration Attempts
- Adding custom origin headers
- Configuring origin request policies
- Testing different cache behaviors
- Creating invalidations and waiting for propagation

### ALB Investigation
- Checking target group attributes
- Verifying listener configurations
- Testing header forwarding manually

### Keycloak Configuration Experiments
- Trying different proxy modes (`edge`, `reencrypt`, `passthrough`)
- Testing various hostname configuration formats
- Attempting to force HTTPS URLs explicitly

## The Revelation: When "Correct" Configuration Doesn't Work

Here's what was fascinating: according to AWS documentation and Keycloak guides, our configuration was correct. CloudFront should forward headers, ALB should preserve them, and Keycloak should recognize the proxy context.

But reality had other plans.

Through extensive testing, we discovered:

1. **CloudFront Origin Custom Headers** were configured correctly but not being forwarded due to cache policy conflicts
2. **ALB Header Preservation** was working, but the headers weren't reaching Keycloak as expected
3. **Keycloak Version Changes** meant that hostname configuration syntax had evolved between versions

## Multiple Failed Solutions

I'll be honest - I provided several solutions that didn't work:

### Failed Attempt #1: Origin Request Policy
```yaml
# We tried using AWS managed policies
Origin-Request-Policy: Managed-AllViewer
```
*Result: Still didn't forward the custom headers properly*

### Failed Attempt #2: Explicit URL Configuration
```yaml
# Keycloak environment variables
KC_HOSTNAME_URL: https://keycloack.example.com
KC_HOSTNAME_ADMIN_URL: https://keycloack.example.com
```
*Result: Keycloak 26.x handles hostname configuration differently than documented*

### Failed Attempt #3: Command Line Arguments
```yaml
command: ["start-dev", "--hostname-url=https://keycloack.example.com"]
```
*Result: "Unknown option: '--hostname-url'" - the parameter doesn't exist*

## The Turning Point: Simplify Everything

After hours of debugging the CloudFront → ALB → Keycloak proxy chain, we had an epiphany: **why make it so complicated?**

The breakthrough came when we asked: "What if we skip all the AWS proxy complexity and just use a simple EC2 instance with SSL?"

## The Solution: Direct SSL Termination

Instead of fighting with proxy headers through multiple AWS services, we implemented a clean, simple architecture:

```
Browser (HTTPS) → EC2 Nginx (443) → Keycloak Container (8080)
```

### The Implementation

**1. Docker Compose with Nginx and Keycloak:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certbot/conf:/etc/letsencrypt:ro
    depends_on:
      - keycloak

  keycloak:
    image: quay.io/keycloak/keycloak:26.2.5
    environment:
      KC_HOSTNAME: keycloack.example.com
      KC_HOSTNAME_STRICT: true
      KC_HTTP_ENABLED: true
      KC_PROXY: edge
    command: ["start-dev", "--hostname=keycloack.example.com", "--proxy=edge"]
```

**2. Nginx Configuration:**
```nginx
# HTTP to HTTPS redirect
server {
    listen 80;
    server_name keycloack.example.com;
    
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
    
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS with proper proxy headers
server {
    listen 443 ssl;
    server_name keycloack.example.com;

    ssl_certificate /etc/letsencrypt/live/keycloack.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/keycloack.example.com/privkey.pem;

    location / {
        proxy_pass http://keycloak:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}
```

**3. Let's Encrypt SSL Certificate:**
```bash
# Get free SSL certificate
docker-compose run --rm certbot certonly \
  --webroot --webroot-path=/var/www/certbot \
  --email your-email@example.com \
  --agree-tos --no-eff-email \
  -d keycloack.example.com
```

## The Final Debugging Challenge

Even with the simplified architecture, we hit one last snag. Keycloak was still generating HTTP URLs, and the error logs showed:

```
Unknown option: '--hostname-url'
Possible solutions: --hostname, --hostname-admin, --hostname-strict
```

The issue? Keycloak 26.x uses different command-line options than what various online tutorials suggest. The working configuration required the correct syntax:

```yaml
command: ["start-dev", "--hostname=keycloack.example.com", "--hostname-strict=true", "--proxy=edge"]
```

## Lessons Learned

### 1. Sometimes Simple is Better
The CloudFront + ALB architecture seemed enterprise-grade and "proper," but it introduced unnecessary complexity. A single EC2 instance with Nginx proved more reliable and easier to debug.

### 2. Documentation Lag is Real
Keycloak's configuration syntax changes between versions, and online tutorials often reference older versions. When troubleshooting, always check the specific version's documentation.

### 3. Proxy Headers are Tricky
The chain of proxy header forwarding (CloudFront → ALB → Application) has many potential failure points. Each service needs proper configuration, and debugging requires testing each link individually.

### 4. Don't Trust "Should Work" Solutions
Just because AWS documentation says CloudFront forwards custom headers doesn't mean your specific setup will work as expected. Always test and verify.

### 5. Cost-Benefit Analysis Matters
Our simple solution costs ~$5/month for the EC2 instance vs ~$30+/month for ALB + CloudFront, with better reliability and easier maintenance.

## The Happy Ending

After implementing the direct SSL approach:

✅ **Mixed content errors**: Gone  
✅ **HTTPS URLs**: Generated correctly  
✅ **SSL certificate**: Free and auto-renewing  
✅ **Maintenance**: Minimal  
✅ **Cost**: Significantly lower  

```javascript
// What we finally achieved:
{
  "serverBaseUrl": "https://keycloack.example.com",  // ✅ HTTPS!
  "adminBaseUrl": "https://keycloack.example.com",
  "authUrl": "https://keycloack.example.com"
}
```

## Key Takeaways for Fellow Developers

1. **Start simple** - Don't over-engineer your SSL termination
2. **Let's Encrypt is fantastic** - Free, reliable, and well-supported
3. **Direct proxy configuration** often works better than multi-service chains
4. **Test incrementally** - Debug one component at a time
5. **Question complexity** - If it's hard to configure, maybe there's a simpler way

## Final Thoughts

This adventure taught me that sometimes the "enterprise" solution isn't the best solution. While AWS managed services have their place, a well-configured EC2 instance with Nginx and Let's Encrypt can be more reliable, cheaper, and easier to maintain than a complex proxy chain.

The next time you're fighting with mixed content errors behind AWS services, consider this: maybe the problem isn't your configuration - maybe it's your architecture.

