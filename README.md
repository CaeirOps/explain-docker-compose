# Docker Compose

## O que é o compose
Como foi visto em nosso post anterior sobre o [Docker](https://blog.4linux.com.br/docker-beginners/), para realizar a execução de um container basta o comando 'docker container run' que o container é criado, porém e se tivermos que executar vários containers de uma única vez e cada um com propósito distinto? Seria muito trabalhoso executar diversas vezes o mesmo comando e ainda incluir os parâmetros necessários, além da criação de rede e volumes se necessário.

É aí que entra o Docker Compose, uma ferramenta instalada separadamente do Docker Engine mas que também faz parte oficialmente do projeto Docker. O Docker Compose é utilizado justamente para facilitar o provisionamento e gerenciamento de multi-containers principalmente em ambientes de desenvolvimento, testes automatizados ou cenário de execução em um único host, bastando apenas um arquivo YAML com as instruções e parametros desejados para os nossos containers que com um único comando conseguimos realizar a execução e/ou atualização de todos eles. Ele ainda realiza o isolamento do ambiente de um conjunto de containers separando-os por nome de "projeto", este nome por padrão é dado o nome do diretório base em que o projeto está sendo executado, porém é possível atribuir um outro utilizando o parâmetro "-p".

## Comandos do Docker Compose
O Docker Compose possui alguns comandos a serem utilizados para realizar toda essa facilidade no provisionamento e gerenciamento dos containers, vejamos então os principais:

 - docker-compose up: Cria e inicia os containers;
 - docker-compose build: Realiza apenas a etapa de build das imagens que serão utilizadas;
 - docker-compose logs: Visualiza os logs dos containers;
 - docker-compose restart: Reinicia os containers;
 - docker-compose ps: Lista os containers;
 - docker-compose scale: Permite aumentar o número de réplicas de um container;
 - docker-compose start: Inicia os containers;
 - docker-compose stop: Para os containers;
 - docker-compose down: Para e remove todos os containers e seus componentes como rede, imagem e volume;

## Estrutura do arquivo
Como já dito, para executarmos nossos containers com o compose é necessário um arquivo YAML que contenha todas as informações e parâmetros que desejamos passar para a execução deles. Por padrão os comandos "docker-compose" procuram um arquivo no diretório corrente nomeado como "docker-compose.yml", porém é possível indicar um outro nome de arquivo e também em um outro local passando o parâmetro "-f". Abaixo podemos observar um exemplo de um arquivo que contém a estrutura dos principais parâmetros utilizados para a execução dos containers via compose:


```yml

version: '3.8'

networks:
  web_network:
    driver: overlay

volumes:
  site_root:

services:
  web-server:
    image: httpd:latest
    ports: "80:80"
    deploy:
      placement:
        constraints:
          - "node.role==worker"
      mode: replicated
      replicas: 2
      resource:
        cpus: "0.50"
        memory: 256M
    restart: always
    networks:
      - web_network
    volumes:
      - site_root:/var/www/html

```

Parece um pouco confuso ao primeiro olhar, mas posso garantir, tudo isso faz muito sentido, rs!! Abaixo iremos detalhar esses principais parâmetros utilizados no exemplo para entendermos melhor o que cada um é responsável. É muito importante ficar atento a endentação das configurações no documento, caso você não esteja familiarizado com o tipo de arquivo YAML, uma endentação incorreta pode invalidar toda configuração passada no arquivo.

### Conceito de serviços
Observando o conteúdo do arquivo YAML mostrado acima, podemos observar que há um bloco de configuração denominado "services", isso se deve por quê o Docker Compose trata todos os containers que desejamos executar como serviços e é dessa forma que devemos referênciá-los no arquivo de configuração. Tratando os containers dessa forma, o compose consegue atribuir o mesmo conjunto de configurações para todos os containers que fizerem parte deste serviço, por exemplo:

 - No caso do arquivo acima temos um parâmetro (que iremos detalhar mais abaixo) chamado "replicas", onde este serve para informar a quantidade de containers "idênticos" desejamos em nosso ambiente, assim o compose irá criar a quantidade de containers informada e todos com a mesma configuração, identificando-os de uma forma parecida com esta: "nome-do-projeto_web-server.1, nome-do-projeto_web-server.2, nome-do-projeto_web-server.n"

O nome do serviço pode ser variado e não há uma regra

### build e image

### ports e expose

### deploy

### restart e restart_policy

### network

### volumes

### environment e env_file

### depends_on

## command e entrypoint

### demonstração

## Onde utilizar?

## Próximos passos

## NOTAS

- Falar da atualização dos containers
- Falar da versão e engine

