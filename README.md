# your-dns
A docker-compose file to provide a secure adblocking DNS server

NOTE: if you are interested in a hosted solution, please take a look at
[nextdns.io](https://nextdns.io). I'm not affiliated with nextdns.io.

## All components in this stack

1. Unbound: This is the actual DNS server that provide DNS-over-TLS
   service at TCP port 853. Unbound will forward DNS request to pihole's
   53 port over UDP.
2. Pihole: Ad blocking DNS server. Pihole forked dnsmasq and provide a
   nice UI to manage the server.
3. Stubby DNS: A DNS stub server, which support forwarding DNS request
   to upstream DNS-over-TLS server. Note Unbound also support forwarding
   request to upstream over TLS, but I was told (can't find the
   reference) Unbound does not reuse TLS connections which is a concern
   to me (my ATT gateway has an internal NAT table with limited # of
   entries).
4. Pomerium: A identity-aware reverse proxy. This allows me to remote
   access PiHole's web UI.
5. Certbot: Free Let's Encrypt certification (required by DNS-over-TLS).
6. Autoheal: Auto-restart container that failed health check.
7. Ouroboros: Auto-pull latest version of each container.

## Run the stack

1. Create a network called `infra_network`
```
    docker network create --subnet 172.30.0.0/16 infra_network
```

2. Create an `.env` file in the directory of `docker-compose.yaml` file
   with the following content:

```
DNS_DOMAIN_NAME=dns.example.com
# Optional, if you want remote access to pihole web UI
# See https://www.pomerium.io/docs/identity-providers.html on detailed
# instruction.
POMERIUM_CLIENT_ID=YOUR_CLIENT_ID
POMERIUM_CLIENT_SECRET=YOUR_CLIENT_SECRET

# Generate two random strings using `head -c32 /dev/urandom | base64`
POMERIUM_SHARED_SECRET=YOUR_RANDOM_STRING
POMERIUM_COOKIE_SECRET=YOUR_RANDOM_STRING
```

3. Configure certbot to generate certificate for your DNS server's
   domain name. This is a tricky part, in order to ease the setup, I
   switched to cloudflare as my name server and uses
   `certbot/dns-cloudflare` image to get certificate via DNS challenge.
   You'll likely need to adjust this part to your own situation.
4. `docker-compose up -d` and you are done :-)

(well I omitted a lot, i.e. you need configuration for
unbound/stubby/pomerium. Until those are ready, please refer to each
project's documentation on how to configure everything.)


## TODO

1. Detailed instruction on how to turn this into a working stack.
2. DNS-over-HTTPS support (Personally not using any device that supports
   it.
