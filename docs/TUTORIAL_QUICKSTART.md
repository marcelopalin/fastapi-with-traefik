# REVERSE PROXY COM TRAEFIK E DOCKER

## Comandos Docker

Parar todos os containers
docker stop $(docker ps -a -q)

Remover todos os contâiners
docker rm $(docker ps -a -q)

Remove all containers
docker rm $(docker ps -a -q)

Remove all docker images
docker rmi $(docker images -q)

## Tutorial

https://doc.traefik.io/traefik/getting-started/quick-start/

O Docker pode ser uma maneira eficiente de executar aplicativos web em produção, mas você pode querer executar vários aplicativos no mesmo host do Docker. Nesta situação, você precisará configurar um proxy reverso, já que você só deseja expor as portas 80 e 443 para o resto do mundo.

O Traefik é um proxy reverso que reconhece o Docker e inclui seu próprio painel de monitoramento ou dashboard. Neste tutorial, você usará o Traefik para rotear solicitações para um container de aplicação. 
Você irá configurar o Traefik para servir tudo através de HTTPS utilizando o Let's Encrypt.

A aplicação é muuuuito simples, é a whoami

https://medium.com/@deepeshtripathi/service-autodiscovery-using-traefik-for-docker-containers-6f3f2ef4f1e1

# Passo 1

Monte o arquivo docker-compose.yml

```s
version: '3'

services:
  reverse-proxy:
    # The official v2 Traefik docker image
    image: traefik:v2.4
    # Enables the web UI and tells Traefik to listen to docker
    command: --api.insecure=true --providers.docker
    ports:
      # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
```

Inicie o Traefik (lembrando que estamos usando sua máquina local e portanto
não temos configurado o DNS para habilitarmos o Let's encripty).

```
docker-compose up -d reverse-proxy
```

Vamos adicionar o serviço Whoami ao docker-compose.yml:

```s
version: '3'

services:
  reverse-proxy:
    # The official v2 Traefik docker image
    image: traefik:v2.4
    # Enables the web UI and tells Traefik to listen to docker
    command: --api.insecure=true --providers.docker
    ports:
      # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
  whoami:
      # A container that exposes an API to show its IP address
      image: traefik/whoami
      labels:
        - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
```

Acesse http://localhost:8080 (cuidado, garanta que seja http)
e você verá o painel do Traefik funcionando.


Agora basta subirmos o serviço em outro container que o Traefik
identificará automaticamente e passará a roteá-lo.

```s
docker-compose up -d whoami
```

Acesse a URL definida no docker-compose.yml [Whoami](http://whoami.localhost/)

```s
Hostname: 7b4112cb2510
IP: 127.0.0.1
IP: 172.19.0.3
RemoteAddr: 172.19.0.2:37476
GET / HTTP/1.1
Host: whoami.localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate, br
Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6
Sec-Ch-Ua: " Not;A Brand";v="99", "Google Chrome";v="91", "Chromium";v="91"
Sec-Ch-Ua-Mobile: ?0
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 172.19.0.1
X-Forwarded-Host: whoami.localhost
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 896e60e6f19c
X-Real-Ip: 172.19.0.1
```

Veja no painel:

http://localhost:8080/dashboard/

http://localhost:8080/api/rawdata