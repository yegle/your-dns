# your-dns
A docker-compose file to provide a secure adblocking DNS server

**NOTE**: if you are interested in a hosted solution, please take a look at
[nextdns.io](https://nextdns.io). I'm not affiliated with nextdns.io.

## Goal

Run a secure DoT (DNS-over-TLS) and DoH (DNS-over-HTTPS) DNS server that
can do ad blocking and hide your DNS query from your ISP.

## Non Goal

Hide your DNS query from upstream recursive DNS server. Why? Because to
me hide my trail from various ISPs (Verizon, ATT, and any other ISPs
behind public WiFis) is more important.

## Privacy Tradeoffs

We are running a DNS forwarder instead of a DNS resolver. Running a
forwarder and connect to upstream DNS over secure connection does hide
your DNS queries from your ISP, but it would also leaks your web history
(in the form of DNS query) to the upstream DNS.

Your web history is always open to your ISP until ESNI is widely
adopted. Even with ESNI, it's still easy for the ISP to learn your web
history based on the IP addresses you connected.

The main benefit of running a forwarder that communicate securely with
upstream DNS is that your ISP won't be able to manipulate your DNS query
results, e.g. hijack the `NXDOMAIN` response to show ads, force traffic
to go through a transparent proxy (with more and more sites offering
HTTPS, this is less of a concern) and so on.

There's a trade off you need to make whether the benefit beats the
reduced privacy. Personally, making it harder for the ISP to learn my web
history is a good enough reason.

## All components in this stack

![overview of components](https://g.gravizo.com/source/svg?https://raw.githubusercontent.com/yegle/your-dns/master/graph.dot)

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
1. [DNS-over-HTTPS](https://github.com/m13253/dns-over-https): A DoH
   server.
1. [Pomerium](https://pomerium.io): An identity-aware reverse proxy. This
   allows me to remote access PiHole's web UI.
   ([reference](https://www.pomerium.io/reference/))
1. [Autoheal](https://github.com/willfarrell/docker-autoheal):
   Auto-restart container that failed health check.
1. [Ouroboros](https://github.com/pyouroboros/ouroboros): Auto-pull
   latest version of each container.

## Prerequisites

1. Install Docker ([how](https://docs.docker.com/v17.12/install/)) and
   `docker-compose` command
   ([how](https://docs.docker.com/compose/install/)).
1. Know how to DNAT from your public IP to the server running the stack.
   Or alternatively if you have IPv6, allow dport=853 access to your
   server.
1. Know how to get a Let's Encrypt certificate for your domain. You need
   a single wildcard certificate if you host both DoH server and pihole
   on the same server.

## Run the stack

The following instruction will run a list of jobs on docker to
DNS-over-TLS service on port 853 and foward your request through PiHole
then to Google DNS.

**NOTE**: if you don't trust Google, please modify `./stubby/stubby.yml` and
specify a different `upstream_recursive_servers`. A list of available
DNS-over-TLS name server is available at
https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Test+Servers.

1. Create a network called `infra_network`. (Why not create the network
   in the compose file? Because you cannot *create* the `default` network
   in compose file, and can only *replace* it with `external`.)
```
    docker network create --subnet 172.30.0.0/16 infra_network
```
2. Modify `.env` file. See the comment in that file for instructions.
3. Use your favorite ACME client to create free certificate from Let's
   Encrypt and save it in `./letsencrypt` directory. If your domain
   name's NS is Cloudflare, the following is an example on how to do it
   within docker:
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

None
