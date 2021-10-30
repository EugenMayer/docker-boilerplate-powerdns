## WAT

PowerDNS based setup to run your private DNS Server for your private Domains

Includes:

- [Dnsdist](https://dnsdist.org/) for routing your DNS request (to recursor or auth)
- [Recursor](https://doc.powerdns.com/recursor/) to recurse and cache outgoing DNS quries
- [Auth(orative) server](https://doc.powerdns.com/authoritative/) for private DNS resolution
- a [GUI PowerDNS Admin](https://github.com/ngoduykhanh/PowerDNS-Admin) to maintain your Authorative-Server entries the convenient way.
- SSL offload (Traefik) to offer SSL for you GUI

You can disable the GUI/Traefik anytime or not start with those at all.

## Production setup / usage

```bash
git clone -b https://github.com/EugenMayer/docker-dns-lb-boilerplate dns
cd dns
vi .env
```

Now be sure to configure the following

- pick `docker-compose-traefik.dns.yml` or `docker-compose-traefik.http.yml`
- Generate all the passwords `PDNS_AUTH_API_KEY`, `GUI_SEED_SECRET_KEY`, and optionally `PDNS_RECURSOR_API_KEY`
- set the GUI domain

```env
COMPOSE_FILE=docker-compose.yml:docker-compose-gui.yml:docker-compose-traefik.dns.yml
DNS_PORT=53
GUI_DOMAIN=dns-admin.mycompany.com
PDNS_RECURSOR_API_KEY=<GENERATE PASSSWORD>
PDNS_AUTH_API_KEY=<GENERATE PASSSWORD>
GUI_SEED_SECRET_KEY=<GENERATE SEED>
# set those 2, if you picked dns challenge
TRAEFIK_ACME_CHALLENGE_DNS_PROVIDER=cloudflare
TRAEFIK_ACME_CHALLENGE_DNS_CREDENTIALS=CF_DNS_API_TOKEN=<YOURTOKEN>
```

Do the `Configure private DNS` step and then

```bash
docker-compose up -d
```

## Testing setup

```bash
git clone -b https://github.com/EugenMayer/docker-dns-lb-boilerplate dns
cd dns
cp .env example .env
```

Do the `Configure private DNS` step and then

```bash
docker-compose up -d
```

## Configure private DNS

Clone your repo as stated above

```bash
mkdir -p dnsdist/custom.d/
vi dnsdist/custom.d/02_private_domain.conf
```

Now enter the private-domain/test-domain you want to offer DNS for. You can have multiple of those statements to support
multiple private domains

```
addAction('intranet.company.net', PoolAction('auths'))
addAction('test.example.net', PoolAction('auths'))
```

For the given domain the servers in pool `auths` are used to resolve. In `dnsdist/01_resolv.conf` those pools are defined
by adding the host `auth` to `auths` and `recursor` to `recursors`

## Configuration / GUI Login

1. First you register a new user with the GUI by accessing `http(s)://<youguidomain>`
2. Login and then configure the PowerDNS `auth backend api` via: `http://auth:8081` and setting the API key using from `PDNS_AUTH_API_KEY`.
3. Create a new `native` domain (your private doman) and any sub entries you like

## Validating the setup

1. Test your private resolution using `dig @127.0.0.1 private.intranet.company.net` or if you are using the testing setup `dig @127.0.0.1 -p 2053 private.intranet.company.net`.
2. Then test your recursor using `dig @127.0.0.1 google.com`

## Advanced

### Disable GUI

If you do not need a GUI (or not anymore), you can disable it (temporary or forever)

```bash
vi .env
COMPOSE_FILE=docker-compose.yml
```

That's it, the GUI and traefik is no disabled
