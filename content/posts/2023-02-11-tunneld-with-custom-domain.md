---
title: "Self-hosting tunnel exposing local ports to the internet with custom domain"
date: 2023-02-11T17:17:00+02:00
draft: false
categories:
  - Learning
---

Although there are some existing solutions to expose local port to the internet, I've decided to use self-hosted solution. I've prefered this over hosted one for few reasons:

1. I wanted to have some pre-configured subdomains
2. I wanted those subdomains to be fixed and/or reserved just for me
3. Custom domain was a nice addition
4. Cost of hosted solution was just too high to justify it.
5. It was just a fun exercise

Idea is simple - anyone accessing address like `some_name.yourdomain.dev` will actually access your dev environment on specific port, eg. `localhost:3000`. Overall cost of that will be 5$ for cheapest linode + domain cost.

## How to

1. Order cheap linode (or droplet in digitalocean or any other vps) to have internet-facing IP address
2. Order new domain (or use any of existing you don't exactly need any more)
3. Redirect domains A records to IP address of just-ordered VPS

```
	yourdomain.dev     A   112.123.132.122
	*.yourdomain.dev   A   112.123.132.122
```

4. Log in to the VPS and download this package: [GitHub - hons82/go-http-tunnel: Fast and secure tunnels over HTTP/2](https://github.com/hons82/go-http-tunnel "{rel='nofollow' target='_blank'}") - it's easiest to get it's precompiled binary from [releases](https://github.com/hons82/go-http-tunnel/releases "{rel='nofollow' target='_blank'}") page
5. Skip generation of server's certificate, use letsencrypt for that (I'm hosting my domains on route53 and following steps assume you're as well, if not, first steps will be completelly different)
   1. Get domain's zone from route53
   2. Create new IAM profile with folliowing policy
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "VisualEditor0",
         "Effect": "Allow",
         "Action": [
           "route53domains:ListDomains",
           "route53:ListHostedZones",
           "route53:ListHostedZonesByName"
         ],
         "Resource": ["*"]
       },
       {
         "Sid": "VisualEditor1",
         "Effect": "Allow",
         "Action": [
           "route53:GetChange",
           "route53:ChangeResourceRecordSets",
           "route53:ListResourceRecordSets",
           "route53domains:GetDomainDetail"
         ],
         "Resource": ["arn:aws:route53:::hostedzone/Z01635552BLOUEU2E835A"]
       }
     ]
   }
   ```
   3. Create new IAM user and attach just-created policy
   4. Create set of credentials, store them in `.aws/config` file
   5. Get certificates for the domain using certbot - install `python3-certbot-dns-route53` package and run `certbot certonly --dns-route53 -d '*.yourdomain.dev'`
   6. Symlink generated certificate files in tunneld directory: `/etc/letsencrypt/live/yourdomain.dev/fullchain.pem` to `server.crt` and `/etc/letsencrypt/live/yourdomain.dev/privkey.pem` to `server.key`
6. Start the `tunneld` package and profit

Overall it shouldn't take you more than 30 minutes to set everything up.
