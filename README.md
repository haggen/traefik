# Traefik for local development with Docker

- 🗺️ Traefik as router with auto discovery for Docker containers.
- ➰ Public loopback hostname for easier integration with APIs.
- 🔒 Local HTTPS with self-signed certificates.

## Getting started

> 📝 You'll need Docker, Docker Compose and OpenSSL installed.

Here's what we're going to do:

1. Clone the repository.
2. Decide on a hostname.
3. Create a new bridge network in Docker.
4. Start the Traefik service.
5. Update your applications configuration.

Start by cloning this repository somewhere on your machine.

```sh
$ git clone https://github.com/haggen/traefik.git
```

Now we'll need a hostname to work with. Containers are going to be routed to sub-domains of the chosen hostname. This hostname must loopback, e.g. like `localhost`, but it should also be publicly available, otherwise APIs, such as Google's or Twitter's, won't be able to talk back to your application.

I've set my own — `*.local.crz.li` — and I encourage you to do the same, but there are other options:

- `*.vcap.me`
- `*.localho.st` — Also works with IPv6.
- `*.local.gd`
- `*.7f000001.nip.io`
- `*.localhost.direct` — [Already has the certificate files](https://get.localhost.direct/).

> 💡 You can, and should, verify that your selected hostname does indeed point to your local IP and isn't just a proxy, which could pose a security breach. e.g. Run `host <hostname>` and make sure `127.0.0.1` is printed on screen.

With a hostname selected, open up `compose.yml` on your editor and change the configured rule of Traefik dashboard. i.e. Update the hostname `traefik.local.crz.li`.

Now we have to create a custom network to connect the containers that are going to be routed by Traefik and start its container.

> ℹ️ If you customize the network name you'll also need to update the `compose.yml` file accordingly.

```sh
$ docker network create traefik
$ docker compose up -d
```

Finally, in the `compose.yml` — or `docker-compose.yml` — file of your application you need to connect it to the new network and add the required labels.

See [example.yml](./example.yml) for reference.

## Provide your own certificates

Traefik has a default certificate but you can provide your own, if you want.

If dont want to bother, 📝 [mkcert](https://github.com/FiloSottile/mkcert) is a nice little tool that generates self-signed certificates with good defaults and automatically configure a CA on your system.

But you can also do it with OpenSSL.

```sh
$ cd traefik
$ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout certs/key.pem -out certs/cert.pem -days 365 -subj '/CN=*.local.crz.li'
```

(Work in progress…)

### Add certificate to your browser

If you didn't use mkcert or you're working in a foreign system, like WSL, your browser will be showing the "Not secure" alert to you. That's because the certificate isn't signed by a CA.

You can sort this out by adding your custom certificate to your browser trusted list. This comes with its own set of risks, so beware.

(Work in progress…)

## License

Apache-2.0 © 2022 Arthur Corenzan
