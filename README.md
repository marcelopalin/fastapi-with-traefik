# Projeto Traefik e FastAPI

https://dev.to/tiangolo/deploying-fastapi-and-other-apps-with-https-powered-by-traefik-5dik

Servidor:
monitor.ampereconsultoria.com.br
168.138.148.71

# Criando a Aplicação

Crie e ative o ambiente virtual e
instale:

```
pip install fastapi "uvicorn[standard]"
```

Rode a aplicação:

```
uvicorn app.main:app --reload
```

# Crie o Dockerfile na raiz do Projeto

```
FROM tiangolo/uvicorn-gunicorn-fastapi:python3.8

COPY ./app /app/
```

Rode os comandos.
No mesmo diretório do arquivo Dockerfile:

```
docker build --tag fastapitraefik .
docker image list
docker run -it --rm  --name myapp -p 5000:80 fastapitraefik
```

Colocando a opção --rm você verá que assim que encerrar o container com Ctrl + C
o container será deletado, basta digitar:

```
docker ps -a
```

```
docker container run <parâmetros> <imagem> <CMD> <argumentos>
```

Os parâmetros mais utilizados na execução do container são:

Parâmetro	Explicação
-d	Execução do container em background
-i	Modo interativo. Mantém o STDIN aberto mesmo sem console anexado
-t	Aloca uma pseudo TTY
--rm	Automaticamente remove o container após finalização (Não funciona com -d)
--name	Nomear o container
-v	Mapeamento de volume
-p	Mapeamento de porta
-m	Limitar o uso de memória RAM
-c	Balancear o uso de CPU

Observe que estamos usando a imagem oficial FastAPI Docker: tiangolo/uvicorn-gunicorn-fastapi:python3.8.

A imagem base oficial do Docker faz a maior parte do trabalho para nós, então só temos que copiar o código dentro dela.

Certifique-se de ter o Docker instalado em seu computador local e no servidor remoto.

# Docker Compose

Estamos usando o Docker Compose para gerenciar todas as configurações. Portanto, instale o Docker Compose localmente e no servidor remoto.

Para evitar que o Docker Compose trave, instale haveged:

```
apt install haveged
```

Detalhes técnicos : Docker Compose usa os geradores de números pseudo-aleatórios internos da máquina. Mas em um servidor em nuvem recém-instalado / criado, pode não ter o suficiente dessa "aleatoriedade". E isso poderia fazer os comandos do Docker Compose travarem para sempre, esperando que "aleatoriedade" suficiente para usar. haveged previne/ corrige esse problema.

Depois disso, você pode verificar se o Docker Compose funciona corretamente.

# Arquivos Docker Compose

Para todas as explicações detalhadas dos arquivos Docker Compose, verifique a gravação de vídeo.

Certifique-se de atualizar os domínios de example.com para usar o seu e o e-mail para se registrar no Let's Encrypt, você receberá notificações sobre os seus certificados expirados nesse e-mail.

Além disso, certifique-se de adicionar os registros DNS corretos para seu aplicativo principal e para o painel do Traefik e atualizá-los nos arquivos Docker Compose de acordo.

Aqui estão os arquivos Docker Compose se você deseja copiá-los facilmente.
**docker-compose.traefik.yml**:

```yml
version: '3'
services:

  traefik:
    # Use the latest v2.3.x Traefik image available
    image: traefik:v2.3
    ports:
      # Listen on port 80, default for HTTP, necessary to redirect to HTTPS
      - 80:80
      # Listen on port 443, default for HTTPS
      - 443:443
    restart: always
    labels:
      # Enable Traefik for this service, to make it available in the public network
      - traefik.enable=true
      # Define the port inside of the Docker service to use
      - traefik.http.services.traefik-dashboard.loadbalancer.server.port=8080
      # Make Traefik use this domain in HTTP
      - traefik.http.routers.traefik-dashboard-http.entrypoints=http
      - traefik.http.routers.traefik-dashboard-http.rule=Host(`traefik.fastapi-with-traefik.example.com`)
      # Use the traefik-public network (declared below)
      - traefik.docker.network=traefik-public
      # traefik-https the actual router using HTTPS
      - traefik.http.routers.traefik-dashboard-https.entrypoints=https
      - traefik.http.routers.traefik-dashboard-https.rule=Host(`traefik.fastapi-with-traefik.example.com`)
      - traefik.http.routers.traefik-dashboard-https.tls=true
      # Use the "le" (Let's Encrypt) resolver created below
      - traefik.http.routers.traefik-dashboard-https.tls.certresolver=le
      # Use the special Traefik service api@internal with the web UI/Dashboard
      - traefik.http.routers.traefik-dashboard-https.service=api@internal
      # https-redirect middleware to redirect HTTP to HTTPS
      - traefik.http.middlewares.https-redirect.redirectscheme.scheme=https
      - traefik.http.middlewares.https-redirect.redirectscheme.permanent=true
      # traefik-http set up only to use the middleware to redirect to https
      - traefik.http.routers.traefik-dashboard-http.middlewares=https-redirect
      # admin-auth middleware with HTTP Basic auth
      # Using the environment variables USERNAME and HASHED_PASSWORD
      - traefik.http.middlewares.admin-auth.basicauth.users=${USERNAME?Variable not set}:${HASHED_PASSWORD?Variable not set}
      # Enable HTTP Basic auth, using the middleware created above
      - traefik.http.routers.traefik-dashboard-https.middlewares=admin-auth
    volumes:
      # Add Docker as a mounted volume, so that Traefik can read the labels of other services
      - /var/run/docker.sock:/var/run/docker.sock:ro
      # Mount the volume to store the certificates
      - traefik-public-certificates:/certificates
    command:
      # Enable Docker in Traefik, so that it reads labels from Docker services
      - --providers.docker
      # Do not expose all Docker services, only the ones explicitly exposed
      - --providers.docker.exposedbydefault=false
      # Create an entrypoint "http" listening on port 80
      - --entrypoints.http.address=:80
      # Create an entrypoint "https" listening on port 443
      - --entrypoints.https.address=:443
      # Create the certificate resolver "le" for Let's Encrypt, uses the environment variable EMAIL
      - --certificatesresolvers.le.acme.email=admin@example.com
      # Store the Let's Encrypt certificates in the mounted volume
      - --certificatesresolvers.le.acme.storage=/certificates/acme.json
      # Use the TLS Challenge for Let's Encrypt
      - --certificatesresolvers.le.acme.tlschallenge=true
      # Enable the access log, with HTTP requests
      - --accesslog
      # Enable the Traefik log, for configurations and errors
      - --log
      # Enable the Dashboard and API
      - --api
    networks:
      # Use the public network created to be shared between Traefik and
      # any other service that needs to be publicly available with HTTPS
      - traefik-public

volumes:
  # Create a volume to store the certificates, there is a constraint to make sure
  # Traefik is always deployed to the same Docker node with the same volume containing
  # the HTTPS certificates
  traefik-public-certificates:

networks:
  # Use the previously created public network "traefik-public", shared with other
  # services that need to be publicly available via this Traefik
  traefik-public:
    external: true
```

docker-compose.yml:
```yml
version: '3'
services:

  backend:
    build: ./
    restart: always
    labels:
      # Enable Traefik for this specific "backend" service
      - traefik.enable=true
      # Define the port inside of the Docker service to use
      - traefik.http.services.app.loadbalancer.server.port=80
      # Make Traefik use this domain in HTTP
      - traefik.http.routers.app-http.entrypoints=http
      - traefik.http.routers.app-http.rule=Host(`fastapi-with-traefik.example.com`)
      # Use the traefik-public network (declared below)
      - traefik.docker.network=traefik-public
      # Make Traefik use this domain in HTTPS
      - traefik.http.routers.app-https.entrypoints=https
      - traefik.http.routers.app-https.rule=Host(`fastapi-with-traefik.example.com`)
      - traefik.http.routers.app-https.tls=true
      # Use the "le" (Let's Encrypt) resolver
      - traefik.http.routers.app-https.tls.certresolver=le
      # https-redirect middleware to redirect HTTP to HTTPS
      - traefik.http.middlewares.https-redirect.redirectscheme.scheme=https
      - traefik.http.middlewares.https-redirect.redirectscheme.permanent=true
      # Middleware to redirect HTTP to HTTPS
      - traefik.http.routers.app-http.middlewares=https-redirect
      - traefik.http.routers.app-https.middlewares=admin-auth
    networks:
      # Use the public network created to be shared between Traefik and
      # any other service that needs to be publicly available with HTTPS
      - traefik-public

networks:
  traefik-public:
    external: true
```

docker-compose.override.yml:
```yml
version: '3'
services:

  backend:
    ports:
      - 80:80

networks:
  traefik-public:
    external: false
```

Comece as pilhas
Existem muitas abordagens para colocar seu código e imagens Docker em seu servidor.

Você poderia ter um sistema de Integração Contínua muito sofisticado. Mas, para este exemplo, usar um simples rsyncseria o suficiente.

Por exemplo:
```
rsync -a ./* deploy-monitor:/home/ubuntu/code/fastapi-with-traefik/ 
```

Em seguida, dentro de seu servidor, certifique-se de criar a rede Docker:

```s
docker network create traefik-public
```

Em seguida, crie as variáveis de ambiente para HTTP Basic Auth.

Crie o nome de usuário, por exemplo:

```s
export USERNAME=admin
```

Crie uma variável de ambiente com a senha, por exemplo:

```s
export PASSWORD=admin
```

Use openssl para gerar a versão "hash" da senha e armazená-la em uma variável de ambiente:

export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)

E agora você pode iniciar a pilha do Traefik Docker Compose:

```s
docker-compose -f docker-compose.traefik.yml up
```
Depois que verificar que está funcionando coloca no modo daemon:

```s
docker-compose -f docker-compose.traefik.yml up -d
```

```s
docker-compose up -f docker-compose.yml -d
```



## Comandos Docker

Parar todos os containers
docker stop $(docker ps -a -q)

Remover todos os contâiners
docker rm $(docker ps -a -q)

Limpe tudo
stop all containers:
docker kill $(docker ps -q)

remove all containers
docker rm $(docker ps -a -q)

remove all docker images
docker rmi $(docker images -q)

Não testei
remove all docker volumes
docker volume ls -qf dangling=true | xargs -r docker volume rm


sudo lsof -i -P -n | grep LISTEN

netstat -tulpn | grep LISTEN

