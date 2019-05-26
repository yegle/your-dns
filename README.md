# your-dns
A docker-compose file to provide a secure adblocking DNS server

**NOTE**: if you are interested in a hosted solution, please take a look at
[nextdns.io](https://nextdns.io). I'm not affiliated with nextdns.io.

## All components in this stack

1. [Unbound](https://nlnetlabs.nl/projects/unbound/about/): A DNS server
   that provide DNS-over-TLS service.
   ([doc](https://nlnetlabs.nl/documentation/unbound/unbound.conf/))
1. [Pihole](https://pi-hole.net): Ad blocking DNS server. Pihole forked
   dnsmasq and provide a nice UI to manage the DNS server.
   ([donate](https://pi-hole.net/donate/))
1. [Stubby](https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Daemon+-+Stubby):
   A DNS stub server, which support forwarding DNS request to upstream
   DNS-over-TLS server. Note Unbound also support forwarding request to
   upstream over TLS, but I was told (can't find the reference) Unbound
   does not reuse TLS connections which is a concern to me (my ATT
   gateway has an internal NAT table with limited # of entries).
   ([doc](https://dnsprivacy.org/wiki/display/DP/Configuring+Stubby))
1. [Pomerium](https://pomerium.io): A identity-aware reverse proxy. This
   allows me to remote access PiHole's web UI.
   ([reference](https://www.pomerium.io/reference/))
1. [Autoheal](https://github.com/willfarrell/docker-autoheal):
   Auto-restart container that failed health check.
1. [Ouroboros](https://github.com/pyouroboros/ouroboros): Auto-pull
   latest version of each container.

## Run the stack

The following instruction will run a list of jobs on docker to
DNS-over-TLS service on port 853 and foward your request through PiHole
then to Google DNS.

**NOTE**: if you don't trust Google, please modify `./stubby/stubby.yml` and
specify a different `upstream_recursive_servers`. A list of available
DNS-over-TLS name server is available at
https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Test+Servers

1. Create a network called `infra_network`. (Why not create the network
   in the compose file? Because you cannot *create* the `default` network
   in compose file, and can only *replace* it with `external`.)
```
    docker network create --subnet 172.30.0.0/16 infra_network
```

2. Create an `.env` file in the directory of `docker-compose.yaml` file
   with the following content:

```
DNS_DOMAIN_NAME=dns.example.com
# All the remaining are optional.
# If you don't want remote access to pihole web UI, remove pomerium from
# docker-compose.yaml file.
# See https://www.pomerium.io/docs/identity-providers.html on detailed
# instruction.
POMERIUM_CLIENT_ID=YOUR_CLIENT_ID
POMERIUM_CLIENT_SECRET=YOUR_CLIENT_SECRET
# Generate two random strings using `head -c32 /dev/urandom | base64`
POMERIUM_SHARED_SECRET=YOUR_RANDOM_STRING
POMERIUM_COOKIE_SECRET=YOUR_RANDOM_STRING
```

3. Use your favorate tool to create free certificate from Let's Encrypt
   and save it in `./letsencrypt` directory. If your domain name's NS is
   on cloudflare, the following is an example on how to do it within
   docker:
```
  certbot:
    image: certbot/dns-cloudflare:latest
    container_name: certbot
    dns: 172.29.1.1
    restart: unless-stopped
    volumes:
      - ./letsencrypt/etc:/etc/letsencrypt
      - ./letsencrypt/var:/var/lib/letsencrypt
      - ./letsencrypt/credentials.txt:/credentials.txt
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"	
```
4. `docker-compose up -d` and you are done :-)

## TODO

1. DNS-over-HTTPS support (Personally not using any device that supports
   it.
