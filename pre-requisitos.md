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



```bash

```
