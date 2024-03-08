# Caddy Proxy


# Caddy Webserver and Reverse Proxy


## In your Homelab

Install in Ubuntu
```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

Install in Rhel
```
dnf install 'dnf-command(copr)'
dnf copr enable @caddy/caddy
dnf install caddy

systemctl status caddy
```

Caddy command
```
caddy
```

# Running Caddy as a Webserver loccaly

```
systemctl stop caddy
systemctl disable caddy

caddy run

# In a second terminal sho the configuratin
curl localhost:2019/config/   # Workes only on the same server
null     # because no config when installed
```



# Add a config
```
vi mysite.json

{
	"apps": {
		"http": {
			"servers": {
				"example": {
					"listen": [":2015"],
					"routes": [
						{
							"handle": [{
								"handler": "static_response",
								"body": "Hello, world!"
							}]
						}
					]
				}
			}
		}
	}
}


```

Then upload it
```
curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @caddy.json
```
Error: Permission denied.
Solution: Maybe you have to change the port to a higher port 4444

curl localhost:444

# You can also check your config
curl localhost:2019/config/

# Fix port permissions issues (low ports)

Solution 1: sudo caddy run
Solution 2: sudo setcap CAP_NET_BIND_SERVICE=+eip $(which caddy)
            caddy run

Check: curl localhost:2019/load -H "Content-Type: application/json" -d @caddy.json




## In the Cloud

Configure a dns name to your vm in the cloud

## Run a webserver on port 80

caddy file-server

## Run a webserver on port 9000

caddy file-server --listen :9000 

In the directory there must be a index.html file


caddy file-server --listen :9000 --browse

# Or

```
mkdir -p /var/www
touch index.html
caddy file-server --listen :9000 --browse --root /var/www
```


## Using the Caddyfile

vi Caddyfile
```
localhost

respond "Hello, world!"
```

## Create a webserver


vi Caddyfile
```
test.devopszone.de
# or localhost when running locally

respond "Hello, I'm test.devopszone.de"

```
caddy run --watch --config Caddyfile
Hint: You get a letsencrypt certificate now allready

## Test locally

vi Caddyfile
```
localhost

respond "Hello, I'm localhost"
```
caddy run --watch --config Caddyfile
You get a self signed certificate automatically
https:// localhost

## Serving your first website

vi Caddyfile
```
test.devopszone.de {

  root * /var/www/test_devopszone_de

  file_server
}
```
caddy stop

mkdir /var/www/test_devopszone_de
cd /var/www/test.devopszone.de
touch index.html

caddy run --watch --config Caddyfile



## Serving multiple websites with subdomains in the cloud vm

test1.devopszone.de
test2.devopszone.de

test with ping ip the ip is set to your vm

vi Caddyfile
```
test1.devopszone.de {  
  respond "Hello, I'm test1.devopszone.de"
}

test2.devopszone.de {  
  respond "Hello, I'm test2.devopszone.de"
}

```


vi Caddyfile
```
test1.devopszone.de {  
  root * /var/www/test1_devopszone_de
  file_server
}

test2.devopszone.de {  
  root * /var/www/test2_devopszone_de
  file_server
}

```
mkdir -p /var/www/test1_devopszone_de
mkdir -p /var/www/test2_devopszone_de
echo "I'm website 1" > /var/www/test1_devopszone_de
echo "I'm website 2" > /var/www/test2_devopszone_de

caddy run --watch --config Caddyfile



## Serving websites on multible ports (locally without a domain)

vi Caddyfile
```
localhost:8080 {
  root * /var/www/test1_devopszone_de
  file_server
}

localhost:9000 {
  root * /var/www/test2_devopszone_de
  file_server
}
```
create index and start 
curl http://localhost:8080
curl http://localhost:9000


## Adding gzip compression

vi Caddyfile
```
test.devopszone.de {
  root * /var/www/test_devopszone_de
  file_server
  encode gzip
}
```

## Configure Caddy with Env variables

export SITE_URL=test.devopszone.de

vi Caddyfile
```
{$SITE_URL} {
  root * /var/www/$SITE_URL
  file_server
}
```

## Caddy Structure

```
{
  servers {}
}


(snippet) {
  respond "Hello, I'm Thomas"
}

localhost {
  import snippet
}

localhost:9999 {
  respond "Hello, I'm localhost"
}


```


## Run a website in Docker

image: caddy:latest
ports:
  - 80:80
  - 443:443
  - 443:443/udp
volumes:
  - ./Caddyfile:/etc/caddy/Caddyfile
  - caddy_data:/data
  - caddy_config:/config
  - ./index.html:/var/www/index.html

Caddyfile
```
test.devopszone.de {
  root * /var/www/
  file_server
}
```


## Dockerfile

vi Dockerfile
```
FROM caddy:latest
COPY index.html /var/www/index.html
COPY Caddyfile /etc/caddy/Caddyfile
```

## Reverse Proxy

#vi Caddyfile
#```
#test.devopszone.de {
#  reverse_proxy localhost:8080
#  reverse_proxy localhost:9000
#}
#```

vi Caddyfile
```
test.devopszone.de {
  reverse_proxy whoami:80 whoami2:80 whoami3:80 {
    lb_policy round_robin
    lb_try_duration 1s
    lb_try_interval 250ms
  }
  
}
```



traefik/whoami:latest
containername: whoami1  # Important because it maps to the name and then with the internal port !!!!!!!!
port: 8080:80

traefik/whoami:latest
containername: whoami2
port: 8081:80

traefik/whoami:latest
containername: whoami3
port: 8082:80

caddy:latest
ports:
  - 80:80
  - 443:443
  - 443:443/udp
volumes:
  - ./Caddyfile:/etc/caddy/Caddyfile
  - caddy_data:/data
  - caddy_config:/config



## Monitoring Caddy

```
{
  servers {
    metrics
  }
}

test.devopszone.de {
  respond "Hello, World!"
}
```
curl localhost:2019/metrics


## Tip

Reverse Proxy simplest

```
:9000

reverse_proxy :9000
```


Or when installed locally
```
vi /etc/caddy/Caddyfile

:80 {
  reverse_proxy :9000
  # reverse_proxy jenkins:9000


}



```














