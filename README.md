# My Traefik template

- 🗺️ Traefik as ingestion for Docker containers with auto discovery.
- ➰ Public loopback hostname for easy development.
- 🔒 HTTPS with Let's Encrypt ot self-signed certificates.

## Getting started

> [!Important]
> You'll need at least Docker and Docker Compose installed.

Here's what we're going to do:

1. Clone the repository.
2. Create configuration file.
3. Create a new bridged network in Docker.
4. Start Traefik.

Start by cloning this repository somewhere on your machine.

```sh
git clone https://github.com/haggen/traefik.git
```

Now we'll need to decide on a hostname.

For testing or development you can use a loopback, like `localhost`, but I like to use sub-domains to route to my containers and `localhost` can't do that.

So I've created my own public domain that loops back — `*.local.crz.li` — and I encourage you to do the same. But if you don't, there are other options:

- `*.vcap.me`
- `*.localho.st` — Also works with IPv6.
- `*.local.gd`
- `*.7f000001.nip.io`
- `*.localhost.direct` — [Already has the certificate files](https://get.localhost.direct/).

> [!Important]
> You can, and should, verify that your selected hostname does indeed loops back and isn't just a proxy, which could pose a security breach. e.g. Run `host <hostname>` and make sure `127.0.0.1` or `::1` (in casse IPv6) is printed on screen.

Once you've decided on the hostname, copy `compose.override.yml.example` to `compose.override.yml`.

```sh
cp compose.override.yml.example compose.override.yml
```

Open it in your editor and change the rule that matches the Traefik's router.

```diff
-      - "traefik.http.routers.traefik.rule=Host(`traefik.local.crz.li`)"
+      - "traefik.http.routers.traefik.rule=Host(`...`) || Path(`/traefik`)"
```

You must also copy `./config/traefik.yml.example` to `./config/traefik.yml` and, at least, change your let's encrypt email.

```sh
cp config/traefik.yml.example config/traefik.yml
```

Now we have to create a bridged network to connect the containers that are going to be routed by Traefik.

```sh
docker network create traefik
```

Finally, we start Traefik.

```sh
docker compose up -d
```

> [!Tip]
> In the `compose.yml` — or `docker-compose.yml` — of your application you'll need to connect to the new network and add the required labels. See [example/compose.yml](./example/compose.yml) for reference.

## Providing your own certificate

Traefik has a default certificate but you can provide your own, if you want.

If you don't want to bother, 📝 [mkcert](https://github.com/FiloSottile/mkcert) is a nice little tool that generates self-signed certificates with good defaults and automatically configure a CA on your system.

> [!Tip]
> A CA is important so your browser trusts the certificate. See [Trusting your own certificate](#trusting-your-own-certificate) for more information.

But you can also do it with OpenSSL.

```sh
cd traefik
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout certs/key.pem -out certs/cert.pem -days 365 -addext "subjectAltName = DNS:*.local.crz.li"
```

Just remember to change the hostname to the one you chose earlier.

### Trusting your own certificate

If you didn't use mkcert or you're working in a foreign system, like WSL, your browser will be showing the "Not secure" alert to you. That's because the certificate isn't signed by a CA.

You can sort this out by adding your custom certificate to your browser trusted list. This comes with its own set of risks, so beware.

(🚧 Work in progress…)

## License

Apache-2.0 © 2022 Arthur Corenzan
