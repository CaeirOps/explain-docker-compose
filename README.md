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
  site_conf:

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
      - /dir/site/html:/var/www/html
      - site_conf:/etc/httpd

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
Assim como na opção 'networks', aqui nós devemos fazer a declaração dos volumes que desejamos criar e referenciar estes depois dentro de cada serviço que irá utilizá-lo. Em caso de volumes do tipo bind, onde nós mapeamos diretórios existentes em nosso host hospedeiro, a declaração fora da configuração do serviço não é necessária, basta informar o caminho absoluto ou relativo(de acordo com o contexto passado) do diretório que deve ser refletido e o seu destino dentro do container.

### environment e env_file
Com essas opções conseguimos definir variáveis de ambientes para utilizar dentro de nossos containers, algumas imagens já possuem algumas variáveis bem úteis que podem ser utilizadas, importante ler a documentação oficial dela caso esteja baixando do Docker Hub por exemplo.

- environment: Com essa opção é possível passar em formato de lista todas as variáveis de ambiente e seus valores que serão utilizadas no serviço;
- env_file: Nesta opção é possível informar um arquivo que será utilizado como fonte de consulta para configurar as variáveis, onde este arquivo deve conter uma variável por linha e seu respectivo valor.

### depends_on
Podemos com esta opção informar que para que um serviço seja iniciado ele depende que outro seja iniciado primeiro, criando uma dependencia. Assim o compose se encarrega de fazer com que o serviço dependente só seja executado após todos os outros seviços declarados aqui estejam em execução. É um caso de uso realizar a utilização desta opção quando se precisa que um serviço de banco de dados por exemplo, seja provisionado e esteja 'up' para que outro serviço possa consumi-lo assim que for executado, caso contrário este não ficará em estado de execução.

## command e entrypoint
Utilizando estas opções é possível alterar o processo principal responsável pela execução e função daquele container. Ambos subistituem o parametrô 'CMD' e 'ENTRYPOINT' padrão da imagem utilizada. É preciso cuidado com estas opções e utilizar somente em caso real de necessidade de alteração do processo principal do container, pois caso este processo por algum motivo seja terminado, o container também entrará em estado de 'exited'.

### Demonstração
MÃO NA MASSA!!
Algumas distribuições podem possuir o pacote do docker-compose em seus repositórios facilitando a instalação através do gerenciador de pacotes, porém a versão quase sempre não estará atualizada de acordo com as releases liberadas, a vantagem é que em caso de atualização de versão o gerenciado de pacotes irá se encarregar de realizar o processo facilmente pra você.

Contudo não é nada complicado realizar a instalção e atualização de forma manual, garantindo assim que temos a ultima release ou a release desejada diretamente do repositório do docker-compose, veremos então como realizar a instalação de forma manual.

> Lembrando que para realizar este laboratório você já deve possuir o docker instalado e permissão para executálo, caso ainda não tenho verifique o [post](https://blog.4linux.com.br/docker-beginners/) que falamos sobre isso.

#### Instalação
Execute o comando abaixo para realizar o download do binário do docker-compose diretamente do repositório ofical do Github:
```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Feito isso, vamos garantir que teremos permissão de execução deste binário, o que por padrão no Linux não é dado quando se cria um arquivo novo devido à umask padrão do SO, assim execute o comando abaixo:
```shell
sudo chmod +x /usr/local/bin/docker-compose
```

Agora, para que nosso usuário para utilizar este binário de forma simples, sem que seja necessário utilizar o caminho absoluto do arquivo para executar o compose, iremos criar um link simbólico para um diretório já conhecido na variável '$PATH' de todos os usuários:
```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

#### Arquivo Compose
Daremos inicio então a criação de nosso arquivo docker-compose, para isso vamos primeiro realizar a criação de um diretório para assim lá dentro realizarmos a construção de nosso ambiente:
```shell
mkdir lab-compose ; cd lab-compose
```

Feito isso vamos dar início aqui à criação do nosso compose file, onde o propósito será iniciar um serviço Wordpress e um serviço de banco de dados MySQL, para termos acessível um site. Assim você pode copiar e colar em seu terminal o todo o comando a seguir que irá criar o compose file já com o conteúdo necessário para a execução:
```shell
cat <<EOF > docker-compose.yml
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: 
EOF
```

Agora com o arquivo criado, basta executarmos o comando do docker compose para iniciar todos esses serviços com as configurações passadas no arquivo, assim execute em terminal o comando a seguir:
```shell
docker-compose up -d
```

Agora você pode testar em seu navegador se o wordpress já está acessível através do endereço "http://127.0.0.1/", onde deverá abrir a tela de configuração do wordpress.
Você também pode realizar atualizações no seu arquivo compose para modificar as configurações de um serviço e sem a necessidade de 'derrubar' todos os serviços para atualizar o ambiente, basta executar o comando 'up' novamente que somente o serviço alterado será atualizado.

Caso queira remover os container e os recursos criados basta executar o comando:
```shell
docker-compose down
```

## Onde utilizar?
Os cenários onde podemos realizar a execução do compose irá depender bastante das necessidades e políticas implantando em seu local de trabalho, porém a maior utilização dele e seu próposito se dá em ambientes de desenvolvimento ou em ambientes onde se tem apenas um 'node' que irá executar os containers.

## Próximos passos
Agora que já entendemos o que é o compose e como utilizá-lo, o próximo passo é aprender sobre o Docker Swarm, orquestrador de containers da Docker e que também faz uso de arquivos de configuração como o compose para executar todos os serviços de seu cluster.

Então nos vemos no próximo post!!
