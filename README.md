# Traefik for local development with Docker

- πΊοΈ Traefik as router with auto discovery for Docker containers.
- β° Public loopback hostname for easier integration with APIs.
- π Local HTTPS with self-signed certificates.

## Getting started

> π You'll need Docker, Docker Compose and OpenSSL installed.

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

Now we'll need a hostname to work with. This hostname must loopback, e.g. like `localhost`. Containers are going to be routed to sub-domains of the chosen hostname. It also must use a valid TLD, otherwise APIs, like Google's, won't accept it.

I've set my own β `*.local.crz.li` β and I encourage you to do the same, but there are other options:

- `*.vcap.me`
- `*.localho.st` β Also works with IPv6.
- `*.local.gd`
- `*.7f000001.nip.io`
- `*.localhost.direct` β [Already has the certificate files](https://get.localhost.direct/).

> π‘ You can, and should, verify that your selected hostname does indeed point to your local IP and isn't just a proxy, which could pose a security breach. e.g. Run `host <hostname>` and make sure `127.0.0.1` is printed on screen.

With a hostname selected, open up `compose.yml` on your editor and change the configured rule of Traefik dashboard. i.e. Update the hostname `traefik.local.crz.li`.

Now we have to create a custom network to connect the containers that are going to be routed by Traefik and start its container.

> βΉοΈ If you customize the network name you'll also need to update the `compose.yml` file accordingly.

```sh
$ docker network create traefik
$ docker compose up -d
```

Finally, in the `compose.yml` β or `docker-compose.yml` β file of your application you need to connect it to the new network and add the required labels.

See [example/compose.yml](./example/compose.yml) for reference.

## Provide your own certificates

Traefik has a default certificate but you can provide your own, if you want.

If you don't want to bother, π [mkcert](https://github.com/FiloSottile/mkcert) is a nice little tool that generates self-signed certificates with good defaults and automatically configure a CA on your system.

But you can also do it with OpenSSL.

```sh
$ cd traefik
$ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout certs/key.pem -out certs/cert.pem -days 365 -addext "subjectAltName = DNS:*.local.crz.li"
```

Just remember to change the hostname to the one you chose earlier.

### Add certificate to your browser

If you didn't use mkcert or you're working in a foreign system, like WSL, your browser will be showing the "Not secure" alert to you. That's because the certificate isn't signed by a CA.

You can sort this out by adding your custom certificate to your browser trusted list. This comes with its own set of risks, so beware.

(π§ Work in progressβ¦)

## License

Apache-2.0 Β© 2022 Arthur Corenzan
