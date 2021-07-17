# WORDPRESS COM DOCKER

https://medium.com/codingthesmartway-com-blog/wordpress-docker-17595d203052

https://medium.com/the-andela-way/how-to-setup-a-new-wordpress-project-with-docker-7f520f817b97

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

Configurar uma instalação local do WordPress envolve várias etapas e às vezes pode ser muito complicado. Normalmente você precisa configurar um servidor web local (wg Apache), configurar o servidor para poder executar código PHP e configurar um banco de dados MySQL. Claro que você pode usar pacotes pré-compilados como MAMP para MacOS ou XAMPP para Windows para obter todos esses componentes em seu sistema. No entanto, a maneira mais fácil de configurar um ambiente WordPress local é usar a conteinerização com o Docker. 

Em um passado não tão distante, a vida como desenvolvedor não era tão divertida quanto é hoje. Muitas tarefas levaram mais tempo do que deveriam e muitas horas foram gastas correndo atrás de bugs que nos deixavam coçando a cabeça, imaginando qual pacote recém-instalado bagunçou nosso ambiente de desenvolvimento local. Assim, forçando-nos nos piores momentos a recomeçar. Mas esses dias acabaram (pelo menos para mim). Adicionar Docker ao meu fluxo de trabalho tornou minha vida como desenvolvedor muito mais sã e meu computador está mais feliz por isso. Não consigo imaginar começar um projeto hoje em dia sem a conteinerização no fundo da minha mente. Se você trabalha em vários projetos de clientes, a conteinerização pode tornar sua vida mais agradável. Pode adicionar mais complexidade e tempo inicialmente, mas isso desaparece à medida que você fica mais confortável com a tecnologia.
Introdução ao Isolamento
Este não é um artigo sobre Docker: não vou entrar em detalhes sobre o que é Docker e seus recursos. No entanto, acho que será útil escrever algumas coisas sobre a tecnologia para que saibamos o que ela tenta resolver e por que é importante.
A maioria dos desenvolvedores que conheço normalmente tem uma única máquina de trabalho, mas eles tendem a trabalhar em vários projetos ao mesmo tempo. Cada um desses projetos possui dependências de software diferentes e conflitantes. Um projeto pode ser compatível apenas com PHP 5.0, enquanto outro está na vanguarda com PHP 7. Isso também pode se estender para a camada de banco de dados, onde diferentes projetos requerem diferentes versões do MySQL .
Existem algumas abordagens que foram usadas antes do Docker. As linguagens de programação tinham ambientes virtuais onde você cria um ambiente virtual por projeto e especifica quais versões de software esse projeto usa. Todas as dependências do projeto são locais para esse projeto; ou seja, os projetos não compartilham pacotes de software entre si. Outra solução popular foi o uso de máquinas virtuais. Uma máquina virtual é um programa de software que atua como um computador separado. Ele também é capaz de executar seus próprios aplicativos e programas como um computador separado.
Agora, para minha abordagem de isolamento favorita: conteinerização. A conteinerização é uma alternativa leve às máquinas virtuais que envolve o encapsulamento de um aplicativo em um contêiner com seu próprio ambiente operacional. Enquanto as máquinas virtuais incluem um sistema operacional inteiro, as tecnologias de conteinerização compartilham um único sistema operacional host e binários, bibliotecas ou drivers relevantes entre os contêineres. A tecnologia conteinerização mais popular é o open-source Docker, criado por Docker, Inc .

```yml
version: '3'
services:
  wordpress:
    image: wordpress:5.7.2
    container_name: wordpress
    restart: always
    volumes:
      - ./wp-content:/var/www/html/wp-content
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wpdb
      WORDPRESS_DB_USER: user
      WORDPRESS_DB_PASSWORD: '@admin2021!'
    ports:
      - 8081:80
      - 443:443
  db:
    image: mysql:8
    container_name: mysql
    restart: always
    command: "--default-authentication-plugin=mysql_native_password"
    environment:
      MYSQL_ROOT_PASSWORD: '@admin2021!'
      MYSQL_DATABASE: wpdb
      MYSQL_USER: user
      MYSQL_PASSWORD: @admin2021!
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      - 3333:80
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORT: '@admin2021!'
```


Execute o seguinte comando para inicializar os serviços:

```s
docker-compose -f docker-compose-wordpress.yml up -d
```

Ao tentar acessar:  http://localhost:8081

será levado para:
http://localhost:8081/wp-admin/install.php