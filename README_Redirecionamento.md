# Passo 1 - Atualizar Pacotes
```
sudo apt update

sudo apt upgrade
```

# Passo 2 - Preparação do Ambiente
```
# No seu host crie uma pasta chamada de configLB
# Dentro dessa pasta crie um arquivo nomeado de default.conf
# Abra o arquivo e copie isso em seu conteúdo

server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}

# No seu host crie uma outra pasta chamada de configNginx
# Dentro dessa pasta crie um arquivo nomeado de nginx.conf
# Abra o arquivo e copie isso em seu conteúdo

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$http_x_real_ip - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

# Passo 3 - Criação dos Containers
```
# Para os passos seguintes serem realizados com sucesso é necessario ter o docker instalado.
# Faça a instalação seguindo o guia da Digital Ocean: Docker install DigitalOcean

# Criação do LoadBalancer
docker run -dit -p 84:80 --name loadbalancer nginx:1.29.3-alpine

# Criação do Nó
docker run -dit --name no1 -v /var/www/html:/usr/share/nginx/html -v /home/user/configLB:/etc/nginx/conf.d nginx:1.29.3-alpine
```

# Passo 4 - Pegando os IP(s) do(s) Nó(s)
```
# O comando abaixo irá abrir o terminal do(s) nó(s)
docker exec -it no1 sh

apk add nano

# Guarde o IP obtido através do comando abaixo
hostname -i
```

# Passo 5 - Configurar LoadBalancer
```
# Esse comando irá abrir o terminal do lb
docker exec -it loadbalancer sh

apk add nano

cd /etc/nginx/conf.d

nano default.conf

# Substitua todo conteúdo de default por isso

# Substitua ipNode pelo IP dos nós
upstream nos {
        server ipNode1;
        server ipNode2;
        server ipNode3;
}

server {
        listen 80;
        location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://nos;
        }
}
```

# Passo 6 - Reinicie os Containers e Teste a Solução
```
docker stop $(docker ps -a -q)
docker start $(docker ps -a -q)

# Abra um navegador e pesquise por: localhost:84
```
