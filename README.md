# your-dns
A docker-compose file to provide a secure adblocking DNS server

NOTE: if you are interested in a hosted solution, please take a look at
<nextdns.io>. I'm not affiliated with <nextdns.io>.

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

## TODO

1. Detailed instruction on how to turn this into a working stack.
2. DNS-over-HTTPS support (Personally not using any device that supports
   it.
