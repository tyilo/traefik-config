# Traefik v2 configuration

My production [Traefik](https://containo.us/traefik/) configuration.

It automatically redirects HTTP to HTTPS for all domains and retrieves TLS certificates for domains using [Let's Encrypt](https://letsencrypt.org/).

## Setup

You probably want to replace `traefik.tyilo.com` in `docker-compose.yml` with your own domain name, where you want to access the Traefik dashboard from.

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

Make sure that docker in swarm mode:

```sh
docker info | grep 'Swarm: inactive' && docker swarm init
``

Deploy Traefik as stack:

```sh
sudo docker stack deploy --compose-file docker-compose.yml traefik
```

And deploy the [example server](example-server):

```sh
cd example-server
sudo docker stack deploy --compose-file docker-compose.yml example-server
```

You can test the HTTP to HTTPS redirection with:

```sh
curl -i http://example.test/ --resolve example.test:80:127.0.0.1
```

```
HTTP/1.1 302 Found
Location: https://example.test/
Date: Wed, 08 Apr 2020 09:04:45 GMT
Content-Length: 5
Content-Type: text/plain; charset=utf-8

Found
```

And test the server with:

```sh
curl -k https://example.test/ --resolve example.test:443:127.0.0.1
```

```
Yay, Traefik seems to work!
```

If you want to access the website or Traefik's dashboard from your browser, you can add the following lines to your `/etc/hosts` file:

```
127.0.0.1 traefik.tyilo.com
127.0.0.1 example.test
```

and then access [https://traefik.tyilo.com/](https://traefik.tyilo.com/) or [https://example.test/](https://example.test/).

You will need to ignore the warning in your browser as you probably haven't obtained a valid TLS certificate for the domains.


## HTTP/TLS passthrough

Sometimes you don't want to redirect HTTP to HTTPS for a specific domain.

In other cases, we don't want Traefik to do TLS termination, because we want our service to handle TLS itself.

[example-passthrough](example-passthrough) shows configuration to do both of these.
To test it do the following:

```sh
cd example-passthrough
./gen_cert
sudo docker stack deploy --compose-file docker-compose.yml example-passthrough
```


HTTP passthrough can be tested with:

```sh
curl -i http://passthrough.test/ --resolve passthrough.test:80:127.0.0.1
```

```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Length: 26
Content-Type: text/html
Date: Wed, 08 Apr 2020 09:15:02 GMT
Etag: "5e8d8ea8-1a"
Last-Modified: Wed, 08 Apr 2020 08:43:20 GMT
Server: nginx/1.17.9

Hi from HTTP passthrough.
```

TLS passthrough can be tested with:

```sh
curl -vk https://passthrough.test/ --resolve passthrough.test:443:127.0.0.1
```

```
...
* Server certificate:
*  subject: CN=passthrough.test
...
> GET / HTTP/1.1
> Host: passthrough.test
> User-Agent: curl/7.69.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.17.9
< Date: Wed, 08 Apr 2020 09:17:48 GMT
< Content-Type: text/html
< Content-Length: 25
< Last-Modified: Wed, 08 Apr 2020 08:43:33 GMT
< Connection: keep-alive
< ETag: "5e8d8eb5-19"
< Accept-Ranges: bytes
<
Hi from TLS passthrough.
* Connection #0 to host passthrough.test left intact
```

Note that the certificate the server uses is the self-signed certificate we just generated,
meaning Traefik didn't touch the TLS traffic.


## Cleaning up

After testing you can remove all docker stacks with:

```sh
sudo docker stack ls --format '{{.Name}}' | xargs sudo docker stack rm
```
