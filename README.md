# Traefik v2 configuration

My production [Traefik](https://containo.us/traefik/) configuration.

## Setup

You probably want to replace `traefik.tyilo.com` in `docker-compose.yml` with your own domain name, where you want to access the traefik dashboard from.

First make sure to create `acme.json` and `usersfile`:

```sh
touch acme.json usersfile
chmod 600 acme.json
```

Then add a user to access the dashboard with:

```sh
htpasswd usersfile <username>
```

## Usage

Run Traefik with docker-compose:

```sh
sudo docker-compose up -d
```

And run the [example server](example-server):

```sh
cd example-server
sudo docker-compose up -d
```

You can test the server with:

```sh
curl -k https://example.tyilo.com/ --resolve example.tyilo.com:443:127.0.0.1
```

If you want to access the website or Traefik's dashboard from your browser, you can add the following lines to your `/etc/hosts` file:

```
127.0.0.1 traefik.tyilo.com
127.0.0.1 example.tyilo.com
```

and then access [https://traefik.tyilo.com/](https://traefik.tyilo.com/) or [https://example.tyilo.com/](https://example.tyilo.com/).

You will need to ignore the warning in your browser as you probably haven't obtained a valid TLS certificate for the domains.