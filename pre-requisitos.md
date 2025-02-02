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
```

