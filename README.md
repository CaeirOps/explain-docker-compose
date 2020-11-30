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

O nome do serviço pode ser variado, geralmente se atribui um nome que corresponda a função daquele serviço, mas não há uma regra para a criação dos nomes.

### build e image
Podemos ver no arquivo que também há uma opção de 'image', essa opção deve ser utilizada para cada serviço que declararmos dentro do arquivo, pois é com o valor desta opção que o Docker entenderá qual imagem de container deve ser utilizada para a construção do container.

É possível ao invés de utilizar a opção 'image', utilizarmos a opção 'build' onde informaremos o contexto e o arquivo (Dockerfile) que possui as instruções para realizar o build de uma imagem para ser utilizada.

Também podemos combinar as duas opções, onde a opção 'build' continuará exercendo seu papel de fornecer os argumentos necessários para o build de uma nova imagem e a opção 'image' será apenas para informarmos qual será o nome dado à imagem que será construída bem como a sua tag.

### ports e expose
Estas opções fazem refência às portas que serão utilizadas para acessar os serviços providos dentro do container, onde a opção 'ports' é utilizada para informarmos a porta do sistema hospedeiro que receberá as requisições e para qual porta deve encaminhar estas requisições para dentro do container, respectivamente como visto no arquivo modelo acima.

Já a opção 'expose' pode ser utilizada para informar quais portas estarão abertas para receber requisições naquele container, porém apenas das redes qual este faz parte, muito utilizada quando queremos que apenas os serviços se comuniquem entre si para consumir ou encaminhar requisições.

### deploy
Há diversos parametros que podem ser utilizados neste setor. As configurações da opção 'deploy' determinam as regras para a execução do serviço, como mostrado no exemplo os principais argumentos utilizados, onde podemos informar em quais nodes de um cluster este serviço poderá ser executado, o método de execução, quantidade de réplicas e limite de recursos computacionais que serão utilizados pelo serviço.

### restart e restart_policy
Com estas opções conseguimos definer o comportamento desejado em caso de queda do container, a ação que deve ser tomada caso o processo responsável pelo container morra. É possível utilizar apenas uma das duas opções por serviço. Na opção 'restart' temos os seguintes argumentos que podem ser utilizados para definir o comportamento:

 - no: Utilizada para caso o container saia do estado de execução o compose não tente executá-lo novamente;
 - always: Utilizada para que sempre que o container sair do estado de execução, independetemente da causa, o compose execute-o novamente;
 - on-failure: Irá tentar executar o container novamente somente caso o código de retorno resulte em uma falha;
 - unless-stopped: Sempre irá garantir que o estado do container seja em execução, ao menos que este seja colocado em estado 'stopped' manualmente ou por algum outro motivo;

Já a opção 'restart_policy' é utilizada dentro do setor 'deploy', onde conseguimos passar algumas informações adicionais como quantidade de tentativas para iniciar o container, tempo de espera para tentar iniciá-lo novamente, tempo máximo decorrido para executar as tentativas de iniciar o container, e a condição que o compose irá utilizar para tentar executar o container novamente podendo ser escolhidas entre 'none', 'on-failure' e 'any'.

### networks
Aqui nesta opção é onde podemos definir as redes que deverão ser criadas para que os serviços façam parte. Para que uma rede seja criada é necessário realizar a declaração dela como mostrado no arquivo exemplo, onde passamos o nome da rede a ser criada, o tipo da rede e driver e até mesmo a subnet.
Feito a declaração dela é necessário referenciá-la na configuração do serviço para este faço uso da rede criada.

### volumes
Assim como na opção 'networks', aqui nós devemos fazer a declaração dos volumes que desejamos criar e referenciar estes depois dentro de cada serviço que irá utilizá-lo. É possível realizar o uso de todos os tipos de volumes que o Docker permite, bastando

### environment e env_file

### depends_on

## command e entrypoint

### demonstração

## Onde utilizar?

## Próximos passos

## NOTAS

- Falar da atualização dos containers
- Falar da versão e engine
- Falar do compose file com o swarm, e opção de build de não funciona
