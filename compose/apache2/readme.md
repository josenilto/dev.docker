## Passos para usar o Dockerfile

1. Crie uma pasta para o seu projeto.
2. Dentro dessa pasta, crie um subdiretório chamado site e coloque os arquivos do seu site lá (por exemplo, um index.html).
3. Salve o Dockerfile na pasta do projeto.
4. No terminal, navegue até a pasta do projeto e execute os seguintes comandos:

```
# Construa a imagem
docker build -t meu-apache .

# Execute o container
docker run -d -p 80:80 meu-apache
```

Agora, você deve conseguir acessar o Apache no seu navegador através de `http://localhost`.<br>
Se precisar de mais alguma coisa ou ajustes, é só avisar!
