### ✅ [Docker Engine](https://docs.docker.com/engine/install/ubuntu/) on Ubuntu

Docker é um de gerenciador de processos de aplicação em containers.   
Os containers são executados isolados podendo executar varias aplicações.

Instalando o Docker Engine | Executing docker install script.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
docker version
```

Realizando o download de imagens

```bash
docker pull ubuntu
docker images
docker run ubuntu
docker ps  //contêinneres em execução
docker container ls
docker ps -a //contÊinneres em stopados
docker container run
docker run ubuntu sleep 10
docker run ubuntu sleep 1500
docker stop [id]
docker run --help
docker container run --help
docker run -it ubuntu
```
Executando aplicações no contêiner

```bash
docker run -dti  ubuntu 
docker exec -it [id ou nome]  /bin/bash

apt update
apt upgrade -y
apt -y install vim

docker exec -it [id ou nome] cat /etc/*release*
```

Excluindo e nomeando conteinêres

```bash
docker stop [id]
docker rm [id]
docker rmi [imagem]

docker run -dti --name Ubuntu-A ubuntu
```

Copiando arquivos para o contêiner

```bash
docker exec -ti Ubuntu-A /bin/bash
docker exec Ubuntu-A mkdir /destino/
rm -R destino/
docker exec Ubuntu-A mkdir ls -l /

vim Arquivo.txt

docker cp arquivo.txt Ubuntu-A:/aula/
docker cp MeuArquivo.txt Ubuntu-A:/destino/
docker exec Ubuntu-A ls /destino -l

apt install -y zip
apt update
apt upgrade
zip MeuZip.zip *.txt

docker cp MeuArquivo.zip Ubuntu-A:/destino/
docker exec -ti Ubuntu-A /bin/bash
cd /destino/
ls
```

Copiando arquivos do contêiner

```bash
docker cp Ubuntu-A:/destino/Meuzip.zip  Zipcopia.zip
```

Tags

```bash
docker run -dti  debian:9
```

Criando um contêiner do MySQL  
https://hub.docker.com/search?badges=official


```bash

docker pull mysql
docker run -e MYSQL_ROOT_PASSWORD=Senha123 --name mysql-A -d -p 3306:3306 mysql
docker exec -it mysql-A bash

mysql -u root -p --protocol=tcp

CREATE DATABASE aula;
show databases;

docker inspect mysql-A

apt -y install mysql-client

mysql -u root -p --protocol=tcp
```

Acessando o contêiner externamente

```bash
CREATE TABLE alunos (
    AlunoID int,
    Nome varchar(50),
    Sobrenome varchar(50),
    Endereco varchar(150),
    Cidade varchar(50)
);

INSERT INTO alunos (AlunoID, Nome, Sobrenome, Endereco, Cidade) VALUES (1, 'Carlos Alberto', 'da Silva', 'Av. que sobe e desce que ninguém conhece', 'Manaus');

```

Parando e reiniciando um contêiner

```bash
docker stop mysql-A
docker start mysql-A

use aula;
select * from alunos;
docker rm mysql-A
```

Montando (mount) um local de armazenamento

```bash
docker run -e MYSQL_ROOT_PASSWORD=Senha123 --name mysql-A -d -p 3306:3306 --volume=/data:/var/lib/mysql mysql

mysql -u root -p --protocol=tcp --port=3306

CREATE TABLE alunos (
    AlunoID int,
    Nome varchar(50),
    Sobrenome varchar(50),
    Endereco varchar(150),
    Cidade varchar(50)
);



INSERT INTO alunos (AlunoID, Nome, Sobrenome, Endereco, Cidade) VALUES (1, 'Carlos Alberto', 'da Silva', 'Av. que sobe e desce que ninguém conhece', 'Manaus');

```

Tipos de mount (bind, named, dockerfile volume)

As montagens Bind são basicamente apenas vincular um determinado diretório ou arquivo do host dentro do contêiner:

Volumes nomeados são que você cria manualmente com comando:

Eles são criados em /var/lib/docker volumes e podem ser referenciados apenas por seu nome.

Digamos que você crei um volume chamada "mysql_data", você pode apenas referenciá-los como o comando:

dockerfile volume 

Tipo de volume que são criados pela instrução VOLUME. Esses volumes também são criados em /var/lib/docker/ volumes. mas não têm um determinado nome. O volume é criado ao executar o contêiner e são úteis para salvar dados persistentes. O desenvolvedor pode dizer onde estão os dados importantes e o que deve ser persistente.

qual devo utilizar?

Depende de você. Se você quiser manter tudo na "área do docker" (/var/lib/docker), você pode usar volumes. Se você deseja manter sua própria estrutura de diretório, você pode usar binds.

> O Docker recomenda o uso de volumes em vez do uso de binds, pois os volumes são criados e gerenciados pelo docker.

```bash
docker run -v /hostdir:/containerdir mysql
docker volume create nome-do-volume
docker run -v mysql_data:/containerdir mysql
```

Exemplos de tipos de mount, na prática

```bash
****bind mount *****

docker run -dti --mount type=bind,src=/opt/teste,dst=/teste debian
docker exec -ti nomes_usado
docker run -dti --mount type=bind,src=/opt/teste,dst=/teste,ro debian //Só ler os dados "ro"

***volumes****

docker volume create data-debian
docker volume ls

cd /var/lib/docker/volumes/data-debian/_data
touch Arquivo-01.txt
touch Arquivo-02.txt
ls
	
docker run -dti --name debian-A --mount type=volume,src=data-debian,dst=/data debian
docker run -dti --name debian-B --mount type=volume,src=data-debian,dst=/data debian
docker exec -ti debian-A bash
docker exec -ti debian-B bash

docker stop debian-A
docker rm debian-A

docker stop debian-B
docker rm debian-B

docker volume rm data-debian
docker volume ls
```

Mount - Conclusão

```bash
docker run -dti --name centos-A --mount type=volume,src=data-centos-vol,dst=/opt/data-centos-vol centos

docker inspect centos-A

docker exec -ti centos-A bash

docker stop centos-A
docker rm centos-A
docker rm -f centos-A //Remover o conteiner direto sem stop

docker volume rm data-centos-vol
docker volume prune //Excluir todos volume que não está em execução

docker container ls
docker container ls -a

```

Exemplo: Apache Contêiner

```bash
docker run  --name apache-A -d -p 80:80 --volume=/data-apache-vol/apache-A:/usr/local/apache2/htdocs/ httpd

docker run  --name apache-A -d -p 80:80 --volume=/var/lib/docker/volumes/data-apache-vol/apache-A:/usr/local/apache2/htdocs/ httpd

docker run  --name php-A -d -p 8080:80 --volume=/var/lib/docker/volumes/data-apache-vol/apache-A:/var/www/html php:7.4-apache

docker run  --name php-A -d -p 8080:80 --volume=/data/php-A:/var/www/html php:7.4-apache

chmod 754 dokuwiki

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8"/>
<title>Exemplo Apache</title>
</head>
<body>
<h1> OK !! Apache funcionando!!!!! </h1>
</body>
</html>

<?php
phpinfo();
?>

```

Exemplo: PHP-Apache Contêiner

```bash
docker pull php:7.4-apache
docker run  --name php-A -d -p 8080:80 --volume=/var/lib/docker/volumes/data-apache-vol/apache-A:/var/www/html php:7.4-apache

```

Limitando memória e CPU

```bash
docker stats php-A
docker update php-A -m 128M --cpus 0.2
docker run --name ubuntu-C -dti -m 128M --cpus 0.2 ubuntu
docker exec -ti ubuntu-C bash

apt update && apt upgrade && apt install stress
stress --cpu 1 --vm-bytes 50m --vm 1 --vm-bytes 50m
stress --cpu 1 --vm-bytes 150m --vm 1 --vm-bytes 100m
```

Informações, logs e processos

```bash
docker info
docker container logs mysql-A
docker container top mysql-A
```

Redes

```bash
docker network ls
docker network inspect bridge
docker exec -ti ubuntu-C bash

apt-get install iputils-ping

docker network create minha-rede
docker run -dit --name Ubuntu-B --network minha-rede  ubuntu
docker run -dit --name Ubuntu-C --network minha-rede  ubuntu

docker exec -ti ubuntu-B bash

docker rm -f Ubuntu-B Ubuntu-C
docker network rm minha-rede

docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=MyPass@word" -e "MSSQL_PID=Express" -p 1433:1433 -d --name=sql mcr.microsoft.com/mssql/server:latest

```

Primeiro Docker File

```bash
docker run -dti --name ubuntu-python ubuntu
docker exec -ti ubuntu-python bash

apt update && apt install python3 nano && apt clean

cd /opt
nano app.py

nome = input("Qual é o seu nome? ")
print (nome)

python3 app.py
docker exec -ti ubuntu-python python3 /opt/app.py

mkdir /images
cd /images

nano dockerfile

FROM ubuntu

RUN apt update && apt install -y python3 && apt clean

COPY app.py /opt/app.py

CMD python3 /opt/app.py

docker build . -t python-ubuntu
dc
```

Criando uma imagem personalizada do APACHE

```bash
mkdir dedian-apache
cd debian-apache

mkdir site
cd site

wget http://site1368633667.hospedagemdesites.ws/site1.zip
unzip site1.zip
rm site1.zip

tar -czf site.tar ./

vim Dockerfile

Dockerfile
==================

FROM debian
RUN apt-get update && apt-get install -y apache2 && apt-get clean

ENV APACHE_LOCK_DIR="var/lock"
ENV APACHE_PID_FILE="var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="var/log/apache2"

ADD site.tar /var/www/html
LABEL description="Apache Webserver 1.0"
VOLUME /var/www/html
EXPOSE 80
ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-D", "FOREGROUND"]

======================

# docker image build -t debian-apache:1.0 .

# docker run  -dti -p 80:80 --name meu-apache debian-apache:1.0
# docker run  -dti -p 8080:80 --name meu-apache2 debian-apache:1.0

docker ps
ip a

```

Utilizando uma imagem personalizada a partir da imagem do Python

```bash

nome = input("Qual é o seu nome? ")
	print (nome)
=====================================

vim Dockerfile

FROM python
WORKDIR /usr/src/app

COPY app.py /usr/src/app
CMD [ "python", "./app.py" ]

docker image build -t app-python:1.0 .
docker images
docker run -ti --name runappl app-python:1.0

```

Criando uma imagem Multistage

```bash
docker pull golang
docker pull alpine

vim app.go
============================================================

package main
import (
    "fmt"
)

func main() {
  fmt.Println("Qual é o seu nome:? ")
  var name string
  fmt.Scanln(&name)
  fmt.Printf("Oi, %s! Eu sou a linguagem Go! ", name)
}

=============================================================
vim Dockerfile

FROM golang as exec
COPY app.go /go/src/app/

ENV GO111MODULE=auto

WORKDIR /go/src/app
RUN go build -o app.go .

FROM alpine
WORKDIR /appexec
COPY --from=exec /go/src/app/ /appexec
RUN chmod -R 755 /appexec
ENTRYPOINT ./app.go

==============================================================

# docker image build -t app-go:1.0 .
# docker run -ti  app-go:1.0
# docker run -ti  --name meuappOK app-go:1.0

```

Realizando o upload de imagens para o Docker Hub

```bash
docker login
docker build . -t nome-de-usuário/my-go=app:1.0
docker push nome-deu-usuário/my-go=app:1.0
```

Registry: Criando um servidor de imagens

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker logout
docker image tag [id] localhost:5000/meu-apache:1.0

curl localhost:5000/v2/_catalog

docker push  localhost:5000/my-go-app:1.0

*****************
nano /etc/docker/daemon.json 

	{ "insecure-registries":["10.0.0.189:5000"] }
*****************	

systemctl restart docker

docker push  localhost:5000/my-go-app:1.0
docker pull  localhost:5000/my-go-app:1.0
```

Definição e instalação

Docker compose é uma ferramenta desenvolvida para ajudar a definir e compartilhar aplicativos com vários contêineres.  
Com om compose, você pode criar um arquivo YAML para defenir os serviços e com um único comando, pode rodar todos os contêineres ou para-los.

```bash
https://www.adminer.org
```

```bash
apt-get install -y docker-compose
```

Docker-compose: Exemplo prático

```bash
vim docker-compose.yml
====================

version: '3.8'

services:
  mysqlsrv:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: "Senha123"
      MYSQL_DATABASE: "testedb"
    ports:
      - "3306:3306"
    volumes:
      - /data/mysql-C:/var/lib/mysql
    networks:
      - minha-rede

  adminer:
    image: adminer
    ports:
      - 8080:8080
    networks:
      - minha-rede

networks: 
  minha-rede:
    driver: bridge


# docker-compose up -d
# docker-compose down

# docker network ls

```

Exemplo PHP-APACHE-MYSQL

```bash
===============================================
vim docker-compose.yml
===============================================

version: "3.7"

services:
  web:
    image: webdevops/php-apache:alpine-php7
    ports:
      - "4500:80"
    volumes:
      - /data/php/:/app

    networks:
      - minha-rede

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: "Senha123"
      MYSQL_DATABASE: "testedb"
    ports:
      - "3306:3306"
    volumes:
      - /data/mysql-C:/var/lib/mysql

    networks:
      - minha-rede

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      MYSQL_ROOT_PASSWORD: "Senha123"
    ports:
      - "8080:80"
    volumes:
      - /data/php/admin/uploads.ini:/usr/local/etc/php/conf.d/php-phpmyadmin.ini

    networks:
      - minha-rede

networks:
   minha-rede:
     driver: bridge

=============================================================
 vim uploads.ini
=============================================================
 
file_uploads = On
memory_limit = 500M
upload_max_filesize = 500M
post_max_size = 500M
max_execution_time = 600
max_file_uploads = 50000
max_execution_time = 5000
max_input_time = 5000

=============================================================
vim index.php
=============================================================
<html>

<head>
<title>Exemplo PHP</title>
</head>

<?php
ini_set("display_errors", 1);
header('Content-Type: text/html; charset=iso-8859-1');

echo 'Versao Atual do PHP: ' . phpversion() . '<br>';

$servername = "db";
$username = "root";
$password = "Senha123";
$database = "testedb";

// Criar conexão


$link = new mysqli($servername, $username, $password, $database);

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

$query = "SELECT * FROM tabela_exemplo";

if ($result = mysqli_query($link, $query)) {

    
    while ($row = mysqli_fetch_assoc($result)) {
        printf ("%s %s %s <br>", $row["nome"], $row["cidade"], $row["salario"]);
    }

    
    mysqli_free_result($result);
}


mysqli_close($link);

?>
</html>

```

Utilizando o docker machine

```bash
docker-machine version
  
docker-machine ls
docker-machine ip nome
docker-machine ssh nome
  
docker-machine create --driver virtualbox nome

--virtualbox-cpu-count "1"
--virtualbox-disk-size "20000"
--virtualbox-memory "1024"

docker-machine env nome
eval $(docker-machine env nome)


docker-machine stop nome
docker-machine start nome

```



```bash

```



```bash

```



```bash

```


docker-compose up: cria e inicia os contêineres;  
docker-compose build: realiza apenas a etapa de build das imagens que serão utilizadas;  
docker-compose logs: visualiza os logs dos contêineres;  
docker-compose restart: reinicia os contêineres;  
docker-compose ps: lista os contêineres;  
docker-compose scale: permite aumentar o número de réplicas de um contêiner;  
docker-compose start: inicia os contêineres;  
docker-compose stop: paralisa os contêineres;  
docker-compose down: paralisa e remove todos os contêineres e seus componentes como rede, imagem e volume.  

git config --global user.email "josenilto@outlook.com"  
git config --global user.name "Josenilto L da Silva"  

sudo usermod -a -G microk8s $USER  
sudo chown -f -R $USER ~/.kube  
su - $USER  
