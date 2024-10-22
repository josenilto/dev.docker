Passos para usar o Dockerfile

1. Estrutura do projeto: Certifique-se de que sua aplicação Java esteja empacotada em um arquivo JAR (por exemplo, minha-aplicacao.jar). <br>
Coloque-o no diretório target do seu projeto.
2. Salve o Dockerfile: Crie um arquivo chamado Dockerfile na raiz do seu projeto.
3. Construa a imagem: No terminal, navegue até a pasta do projeto e execute:

```
docker build -t minha-aplicacao .
```

4. Execute o container: Após a imagem ser construída, você pode rodar o container com:

```
docker run -d -p 8080:8080 minha-aplicacao
```

Agora, você deve conseguir acessar sua aplicação Java através de `http://localhost:8080`.<br>
Se precisar de mais alguma ajuda ou ajustes, é só avisar!



