## Docker

- Docker: Ferramenta para a criação que facilita a criação de containers
- Container: um ambiente isolado
- Imagens: Instruções para criação de container

## Exemplo de dockerfile

```dockerfile
# Nome da imagem, dipónivel em https://hub.docker.com/_/node
FROM node

# Local de armazenamento do projeto
WORKDIR /usr/app

# Copiar arquivo para o diretorio de trabalho
COPY package.json ./

# Instalar dependencias
RUN npm install

# Copiar todo o resto para o diretorio de trabalho
COPY . ./

# Portal do projeto
EXPOSE 3333

# Executar o projeto
CMD ["npm", "run", "dev"]
```

## Comandos docker

- Criar imagem
  - ```bash
    docker build -t <project_name> <local_dockerfile>
    # Exemplo
    docker build -t app .
    ```
- Ver imagens criadas
  - ```bash
    docker image ls
    ```
- Ver imagens em execucao
  - ```bash
    docker ps
    ```
- Executar uma imagem no docker
  - ```bash
    # Container roda em um ambiente isolado com um ip diferente o -p indica o mapeamento das portas entre as duas máquinas, a sua e a do container
    docker run -p <port_your_machine>:<port_container> <image_name|image_id>
    # Exemplo
    docker run -p 3333:3333 app
    ```
- Abrir container em terminal
  - ```bash
    docker exec -it <container_name> sh
    ```
- Para execução de container
  - ```bash
    docker stop <container_id>
    ```
- Remover container
  - ```bash
    docker rm <container_id>
    ```
- Excluir todas as imagens
  - ```bash
    docker image prune -a
    ```
- Limpar cache de builder
  - ```bash
    docker builder prune
    ```
- Ver logs
  - ```bash
    docker logs <container_name> -f
    ```

## Docker compose

- OBS: `volumes` in docker-compose.yml não utiliza o arquivo `.dockerignore` ou seja, a pasta node_modules não será ignorada!

Ferramenta que é usada como gerenciador de containers

- Subir containers
  - ```bash
      docker-compose up
    ```
- Para execução de container
  - ```bash
    docker-compose stop
    ```
- Remover container
  - ```bash
    docker-compose down
    ```
