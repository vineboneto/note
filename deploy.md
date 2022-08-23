## Deploy na aws

> Criação de usuário

- Criar usuário
  - `sudo adduser <your_user>`
- Dar direitos de sudo ao usuário
  - `sudo usermod -aG sudo <your_user>`
- Conectar ao usuário
  - `sudo su - <your_user>`

> Configurar ssh dentro do linux para conseguir sempre acessar da sua máquina

- Criar pasta uma pasta chama `.ssh` em `~`
- Dar os direitos de leitura e escrita dessa pasta com `chmod 700 .ssh/`
- Em `.ssh` criar arquivo chamado `authorized_keys`
- Dar os direitos de leitura e escrita desse arquivo com `chmod 600 authorized_keys`
- Em seu computador local instalar `ssh` [link](https://slproweb.com/products/Win32OpenSSL.html) ou tente `choco install openssh`
- Executar `ssh-keygen` para gerar uma chave publica
- Em `~/.ssh/id_rsa.pub` terá a sua chave ssh
- Executar `cat ~/.ssh/id_rsa.pub` para exibir chave ssh pública
- Colar dentro de `authorized_keys` a chave ssh gerada
- Reiniciar serviço ssh `sudo service ssh restart`
- Para conectar na instância basta agora executar `<your_user>@<public_ip_of_server>`

> Configuração linux

- Atualizar pacotes do linux
  - `sudo apt update`
- Instalar `nodejs lts`
- Instalar `docker`
- Instalar `docker-compose`
- Instalar o `yarn`

> Configuração SSH github

- Em seu computador local instalar `ssh` [link](https://slproweb.com/products/Win32OpenSSL.html) ou tente `choco install openssh`
- Gerar uma key ssh `ssh-keygen` em `~/.ssh`
- Executar `ssh-keygen` para gerar uma chave publica
- Em `~/.ssh/id_rsa.pub` terá a sua chave ssh
- Executar `cat ~/.ssh/id_rsa.pub` para exibir chave ssh pública
- Copiar a chave ssh e colar em `Github -> Settings -> SSH and GPG keys -> new SSH key`

> Configurar github actions

- Gerar uma key ssh com `ssh-keygen -f <your_ssh-name>` em `~/.ssh` em seu computador
  - Ex: `cd ~/.ssh/ ; ssh-keygen -f github-actions`
- Executar `cat ~/.ssh/github-actions` para exibir chave ssh **pública**
- Em `/etc/ssh/sshd_config`

  -

  ```bash
  PubkeyAuthentication yes
  PubkeyAcceptedKeyTypes=+ssh-rsa
  ```

  - `sudo service ssh restart`

- Copiar chave ssh pública e colar dentro de `~/.ssh/authorized_keys` do seu **servidor**

  - Ex:
    - `cat >> authorized_keys`
    - _colar chave pública_
    - `ctrl+d` para salvar
    - OBS: Dessa forma será concatenado as chaves caso já exista alguma em `authorized_keys`

- Executar `cat ~/.ssh/github-actions` para exibir chave ssh **privada**
- Copiar chave ssh privada e ir para seu projeto no github

  - Em `settings -> Secrets -> Actions -> New repository secret` configurar variáveis secrets
  - Ex:
    ```bash
      SSH_HOST: <your_public_ip>
      SSH_SECRET: <your_ssh_secret>
      SSH_USERNAME: <your_user_of_server>
      SSH_PORT: 22 # Default
    ```

- Em seu projeto, acessar a aba `Actions -> New workflow -> set up a workflow yourself`
  - Nova pasta `.github/workflows/main.yml` será criada
- Arquivo `workflow`

  ```yml
  name: CI # Qualquer nome
  on:
    push:
    branches: [ "main" ]
    pull_request:
    branches: [ "main" ]

    workflow_dispatch:

  jobs:
    build:
    runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v3

          # Configurando nodejs
          - name: Setup Nodejs
            uses: actions/setup-node@v2
            with:
              node-version: 16.x
          - name: Install dependencies
            run: yarn

          - name: Build
            run: yarn build

          # Enviar arquivo para seu servidor
          - uses: appleboy/scp-action@master
            with:
              host: ${{ secrets.SSH_HOST }}
              username: ${{ secrets.SSH_USER }}
              port: ${{ secrets.SSH_PORT }}
              key: ${{  secrets.SSH_KEY }}
              source: "., !node_modules" # Arquivo enviados
              target: "test" # Nome da pasta gerada
  ```

> Configurar Nginx

- Instalar nginx
  - `sudo apt update`
  - `sudo apt install nginx`
  - `sudo service nginx start`
- Abrir local nginx
  - `cd /etc/nginx/sites-available`
- Colar seu arquivo seu arquivo `<your_project_name>`

  - Ex

  ```nginx
  server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location / {
      proxy_pass http://localhost:3333;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
    }
  }
  ```

- Criar um link simbólico `sudo ln -s /etc/nginx/sites-available/<your_conf_file> /etc/nginx/sites-enabled/<your_conf_file>`
- Remover arquivo `default` em `sites-available` e `sites-enabled`
- Restart serviço `sudo service nginx restart`

**Dois proxy reverso**

- Crie um arquivo de `nginx.conf` para cada app em sites-available
- Ex

  - domain-one.conf

  ```nginx
  server {
    listen 80;
    listen [::]:80;

    server_name domain-one.com;

    location / {
      proxy_pass http://localhost:3333;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
    }
  }
  
  ```
  - Criar um link simbólico `sudo ln -s /etc/nginx/sites-available/domain-one.conf /etc/nginx/sites-enabled/domain-one.conf`

  - domain-two.conf

  ```nginx
  server {
    listen 80;
    listen [::]:80;

    server_name domain-two.com;

    location / {
      proxy_pass http://localhost:3333;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
    }
  }
  ```

  - Criar um link simbólico `sudo ln -s /etc/nginx/sites-available/domain-one.conf /etc/nginx/sites-enabled/domain-two.conf`

> Configurar Certificado https

Acesse (certbot)[https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal]

- Siga as instruções de instalação de ubuntu nginx
