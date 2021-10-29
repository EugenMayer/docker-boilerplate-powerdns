## WAT

PowerDNS based setup to run your private DNS Server for your private Domains, including recursion.
It also includes PowerDNS Admin to maintain your Authorative-Server entries the convinient way.

The GUI is behind traefik offering you either `http` or `dns` based ssl offloading.

## Production

```
git clone -b https://github.com/EugenMayer/docker-dns-lb-boilerplate dns
cd dns
vi .env
```

Now be sure to configure the following

- pick `docker-compose-traefik.dns.yml` or `docker-compose-traefik.http.yml`
- generate all the passwords and the seed
- set the GUI domain

```
COMPOSE_FILE=docker-compose.yml:docker-compose-gui.yml:docker-compose-traefik.dns.yml
DNS_PORT=53
GUI_DOMAIN=pdngui.lan
PDNS_RECURSOR_API_KEY=<GENERATE PASSSWORD>
PDNS_AUTH_API_KEY=<GENERATE PASSSWORD>
GUI_SEED_SECRET_KEY=<GENERATE SEED>
# set those 2, if you picked dns challenge
TRAEFIK_ACME_CHALLENGE_DNS_PROVIDER=cloudflare
TRAEFIK_ACME_CHALLENGE_DNS_CREDENTIALS=CF_DNS_API_TOKEN=<YOURTOKEN>
```

Do the `Configure private DNS` step and then

```
docker-compose up -d
```

## Testing

```
git clone -b https://github.com/EugenMayer/docker-dns-lb-boilerplate dns
cd dns
cp .env example .env
```

Do the `Configure private DNS` step and then

```
docker-compose up -d
```

## Configure private DNS

Clone your repo as stated above

```
mkdir -p recursor/custom.d/
vi recursor/custom.d/toauth.conf
```

Now enter the test-domain you want to offer DNS for. You can have multiple of those statements, be sure to use `+=`

The IP is right until you did not reconfigure `NETWORK`

```
forward-zones=intranet.company.net=172.21.0.226:53
```

## Advanced

### Disable GUI

If you do not need a GUI (or not anymore), you can disable it (temporary or forever)

```
vi .env
COMPOSE_FILE=docker-compose.yml
```

That's it, the GUI and traefik is no disabled

### Networking
If you have a collision with the docker-network used in the stack, you can change it in the `.env` file using

```
NETWORK=10.10.0
```

Be sure to adjust your `forward-zones` if you do so, so to `10.10.0.226`
