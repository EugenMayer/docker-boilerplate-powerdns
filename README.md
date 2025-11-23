## WAT

PowerDNS based setup to run your private DNS Server for your private Domains, including recursion.

## Production

```bash
git clone -b https://github.com/EugenMayer/docker-dns-lb-boilerplate dns
cd dns
vi .env
```

Now be sure to configure the following

- generate all the passwords and the seed

```env
COMPOSE_FILE=docker-compose.yml
DNS_PORT=53
PDNS_RECURSOR_API_KEY=<GENERATE PASSSWORD>
PDNS_AUTH_API_KEY=<GENERATE PASSSWORD>
```

Do the `Configure private DNS` step and then

```
docker compose up -d
```

### Access webui / apis

- Just go to `http://localhost:48081` for the auth webapi
    - user: any value
    - password: your `$PDNS_AUTH_API_KEY` value, by default `auth_secret` (see your .env file)

- Just go to `http://localhost:38082` for the recursor webapi
    - user: any value
    - password: your `$PDNS_RECURSOR_API_KEY`, by default `recursor_secret` (see your .env file)


## Testing

```bash
git clone -b https://github.com/EugenMayer/docker-boilerplate-powerdns dns
cd dns
cp .env example .env
docker compose up
```

This starts the default dev server with

auth:
- auth-webapi on 38081 with password `auth_secret` (any user)
- dns port: 3083

recursor:
- recorsor-webapi on 48082 with password `recursor_secret` (any user)
- dns port: 4053

dnsdist:
- dnsdis-webapi on 58083 with password `auth_secret` (any user)
- dns port: 5083

With this setup, you will use 1.1.1.1 as a recursor upstream, except for all hostnames under `test.local` - those
are resolved via the `auth` server and whatever dns entries you have there.

This means when you run those queries

... against the dindist (our front dsn lb / router)
```bash
dig @localhost -p 5053 google.de
# this will query dnsdist service- since it does not mach test.local, it will forward the query to our recursor. This one
# uses 1.1.1.1 and thus the query will return the public google ip - answered by 1.1.1.1

dig @localhost -p 5053 test.local
# this will query dnsdist service - since this matches test.local it will forward our query to auth
# if you did not yet create any dns entries for test.local here (e.g. with OctaDNS) it will not answer - the answer
# will come from the auth server, not 1.1.1 though
# this happens due to our config at examples/custom.d/dnsdist/02_private_domain.conf
```

... against the recursor
```bash
dig @localhost -p 4053 google.de
# query the recursor service, which wil try to resolve via 1.1.1.1 and will return the public google ip - answered by 1.1.1.1

dig @localhost -p 5053 test.local
# query the recursor service, which wil try to resolve via 1.1.1.1 and will return nothing - answered by 1.1.1.1
# so without dnsdist asking the recursor will always lead to using the public DNS
```

... against the auth
```bash
dig @localhost -p 3053 google.de
# query the auth service, which wil try to resolve it and fail, since it has no entry for it - answered by auth service

dig @localhost -p 3053 test.local
# query the auth service, which wil try to resolve it and maybe succeed, if you have an entry for test.local - answered by auth service
```
This give you an idea, what we have what for - and that dnsdist should be what you expose to the public.

## Configure private DNS

Clone your repo as stated above

```
mkdir -p dnsdist/custom.d/
vi dnsdist/custom.d/02_private_domain.conf
```

Now enter the test-domain you want to offer DNS for. You can have multiple of those statements.
You can have multiple entries as here, or just one

```
addAction('intranet.company.net', PoolAction('auths'))
addAction('test.example.net', PoolAction('auths'))
```

