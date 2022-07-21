# Observabilidade: coletando métricas de uma aplicação com Prometheus
iniciado em 12/07/2022

terminado em 

[certificate]() 

- [Observabilidade: coletando métricas de uma aplicação com Prometheus](#observabilidade-coletando-métricas-de-uma-aplicação-com-prometheus)
  - [Apresentaçao](#apresentaçao)
    - [Aplicabilidade](#aplicabilidade)
    - [Observabilidade](#observabilidade)
  - [Monitoramento](#monitoramento)
    - [Metricas](#metricas)
  - [Configurando o ambiente](#configurando-o-ambiente)
  - [Externalizando métricas com Actuator](#externalizando-métricas-com-actuator)
  - [Expondo métricas para o Prometheus](#expondo-métricas-para-o-prometheus)
  - [Métricas da JVM](#métricas-da-jvm)
    - [Memória](#memória)
    - [Logs](#logs)
    - [Hikaricp -- Conexoes e Banco de dados](#hikaricp----conexoes-e-banco-de-dados)
    - [Process](#process)
    - [JVM](#jvm)
    - [CPU](#cpu)
    - [JVM Memory](#jvm-memory)
    - [hikarycp -- tempo para aquisicao de conexao](#hikarycp----tempo-para-aquisicao-de-conexao)
    - [JVM Threads](#jvm-threads)
    - [SLA](#sla)
    - [JVM classes](#jvm-classes)
    - [timeout de conexao com o database](#timeout-de-conexao-com-o-database)
    - [tempo de criaçao de uma conexao com o database](#tempo-de-criaçao-de-uma-conexao-com-o-database)
    - [utilizaçao de CPU](#utilizaçao-de-cpu)
  - [Métricas personalizadas](#métricas-personalizadas)
  - [Preparando a API para a conteinerização](#preparando-a-api-para-a-conteinerização)
  - [Subindo a stack com API e Prometheus](#subindo-a-stack-com-api-e-prometheus)
  - [Conhecendo as configurações do Prometheus](#conhecendo-as-configurações-do-prometheus)
    - [Graph](#graph)
    - [Alerts](#alerts)
    - [Status](#status)
    - [Help](#help)
    - [Prometheus configuration em desenv](#prometheus-configuration-em-desenv)
    - [Prometheus.yaml](#prometheusyaml)
      - [scrape interval](#scrape-interval)
      - [Prometheus job](#prometheus-job)
  - [Subindo o cliente para consumo da API -- automatizado](#subindo-o-cliente-para-consumo-da-api----automatizado)
    - [Pasta client](#pasta-client)
    - [Client-forum-api](#client-forum-api)
  - [A anatomia de uma Métrica](#a-anatomia-de-uma-métrica)
    - [metric name](#metric-name)
    - [labels](#labels)
    - [sample](#sample)
    - [Tipos de Dados no Prometheus](#tipos-de-dados-no-prometheus)
      - [Instant Vector](#instant-vector)
      - [Range Vector](#range-vector)
      - [Scalar](#scalar)
      - [String](#string)
        - [Subquery](#subquery)
  - [Tipos de Métricas](#tipos-de-métricas)
    - [Counter](#counter)
    - [Gauge](#gauge)
    - [Histogram](#histogram)
      - [histogram quantile](#histogram-quantile)
    - [Summary](#summary)
      - [Quantiles configuraveis](#quantiles-configuraveis)
  - [Seletores, Modificadores e Funções](#seletores-modificadores-e-funções)
      - [seletor de negação (!=)](#seletor-de-negação-)
      - [seletor OU LOGICO (=~)](#seletor-ou-logico-)
      - [Aglutinaçao de resultados](#aglutinaçao-de-resultados)
    - [funçoes](#funçoes)
      - [funçao increase()](#funçao-increase)
      - [funçao sum()](#funçao-sum)
      - [funçao irate()](#funçao-irate)
- [Monitoramento: Prometheus, Grafana e Alertmanager](#monitoramento-prometheus-grafana-e-alertmanager)
  - [Subindo o Grafana](#subindo-o-grafana)
    - [configurar um datasource](#configurar-um-datasource)
    - [folders](#folders)
    - [dashboards](#dashboards)
  - [Variáveis e métricas Uptime e Start Time](#variáveis-e-métricas-uptime-e-start-time)
    - [variaveis](#variaveis)
    - [dashboard UPTIME](#dashboard-uptime)
    - [dashboard START TIME](#dashboard-start-time)
  - [Métricas Logback e JDBC Pool](#métricas-logback-e-jdbc-pool)
    - [dashboard Logback (LOGS)](#dashboard-logback-logs)
    - [dashboard JDBC pool (connections)](#dashboard-jdbc-pool-connections)
  - [Métricas Logged Users e Auth Errors](#métricas-logged-users-e-auth-errors)
    - [dashboard Logged users](#dashboard-logged-users)
    - [Auth Errors](#auth-errors)
  - [Métricas Connection State e Connection Timeout](#métricas-connection-state-e-connection-timeout)
    - [Connection State](#connection-state)
    - [Connection Timeout](#connection-timeout)
  - [Métricas Error 500 e Error Rate](#métricas-error-500-e-error-rate)
    - [error 500](#error-500)
    - [Error rate](#error-rate)
  - [Métricas Total Requests e Response Time](#métricas-total-requests-e-response-time)
    - [Total Requests](#total-requests)
    - [Response time](#response-time)


## Apresentaçao
Sejam bem-vindos ao vídeo de apresentação do curso "Observabilidade: coletando métricas de uma aplicação com Prometheus". 

O conteúdo abordado neste vídeo será a aplicabilidade do curso, o que é observabilidade e monitoramento, o que são métricas e qual é o cenário abordado no curso.

### Aplicabilidade
este curso se destina a pessoas desenvolvedoras, pessoas de operações de infraestrutura, pessoas de DevOps e pessoas de engenharia de confiabilidade de sites. 

### Observabilidade

**A observabilidade consiste em acompanhar o estado de execução de um sistema através da externalização de seu comportamento em tempo de execução**. 

Ou seja, você deve tornar sua aplicação observável para uma fonte externa e o que deve ser observável da sua aplicação são os comportamentos que importam realmente para a experiência do usuário final.


## Monitoramento

**O monitoramento consiste em acompanhar o estado de um sistema através de eventos registrados e executar ações como resposta a esses eventos**. 

Essas respostas podem ser derivadas de ações reativas e proativas.

Logicamente, a observabilidade faz parte do escopo de monitoramento. 

Hoje, no cenário que temos de aplicações complexas e distribuídas, não é possível mais ter um monitoramento efetivo sem a observabilidade.

Então ela é fundamental para você entender como seu sistema está se comportamento diante do consumo.

Agora, vamos entender o que são as métricas, porque se estamos lidando com observabilidade e monitoramento, temos que ter algum indicador para trazer os dados importantes.

### Metricas

**Uma métrica é basicamente um indicador de nível de serviço coletado dentro de uma série temporal**. 

Métricas serão utilizadas para medir:
* performance, 
* disponibilidade, 
* saturação (o uso além daquele previsto dos recursos), 
* erros e 
* qualquer informação útil para o negócio.

Entendemos, então, que a métrica é uma **medição em uma janela de tempo que foca em uma propriedade do seu sistema**. 

Uma métrica sempre possui um objetivo específico. 

Para cada ponto de atenção do seu sistema, deve existir uma métrica correspondente, o que significa que tiraremos métricas da infraestrutura, da execução da aplicação e também de regras de negócio.

Faremos isso por meio da instrumentação. 

Então, teremos informações úteis para o time de desenvolvimento, para a operação de infraestrutura e para a área de negócios. 

Entendendo os conceitos de observabilidade, monitoramento e métricas, podemos avançar para o cenário do curso.

![alt text](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/1.PNG)

Aqui mostro um diagrama que mostra o ambiente que utilizaremos. 

Esse ambiente sobe uma stack do Docker Compose, então o pré-requisito para esse curso é ter o Docker e o Docker Compose instalado.

Também será necessário o Java — versão 1.8, o OpenJDK 1.8 — e recomendo a utilização de alguma IDE (Integrated development environment, em português, ambiente de desenvolvimento integrado) que possa trazer dependências de forma automatizada. 

Vamos falar agora sobre a jornada do nosso usuário para consumir essa aplicação. 

temos um cliente sintético (que representa nossos usuários, que é um contêiner) que executará um acesso não-previsível a essa aplicação, gerando tanto acessos em rotas com êxito quanto com erros — de autenticação, logins, buscas por conteúdo inexistente. Mas a maioria desses acessos terá êxito.

Esse cliente inicia sua jornada passando por um proxy reverso que é um contêiner nginx e que fará o redirecionamento para as rotas dessa API.

A API tem seu conteúdo armazenado em um cache que é um contêiner rodando o Redis. 

Logicamente, na subida da aplicação, esse conteúdo não estará "cacheado", então o primeiro acesso não terá um cache hit e acabará indo ao contêiner que é a base de dados (MySQL), retornando o conteúdo para o cliente e, então, armazenando em cache.

Os endpoints tem uma passagem de parâmetros que busca por um ID específico, nessa rota principal que se chama "tópicos", e ela busca um ID de tópicos. Essas consultas sempre vão "bater" no database.

Além disso, existe a autenticação que também validará, através de suas regras de negócio, o que for inserido pelo usuário com dados contidos no database. Ademais, temos a função de delete da API, mas não será utilizada. **O foco será específico sobre a instrumentação e das métricas.**

Então, nesse primeiro passo, o que faremos é **instrumentar essa API**, **externalizar suas métricas com seu comportamento** e subir o Prometheus que está na região inferior desse nosso diagrama, na nossa stack de monitoramento.

O Prometheus, então, será utilizado para entendermos quais são os tipos de métricas default que ele utiliza e começarmos nossa imersão na linguagem PromQL.

## Configurando o ambiente

Nessa aula, vamos configurar o ambiente.

Vou começar importando o projeto. 

Estou com o Eclipse aberto, ele é uma dependência para que você siga o capítulo 1 sem nenhum problema, mas isso não significa que você não possa utilizar outra IDE. 

No painel à esquerda, vou em “Import Projects...” e vou procurar por um projeto Maven já existente.

Dentro do prometheus-grafana, você vai no subdiretório app, em que existe um arquivo com o XML que o Eclipse vai reconhecer automaticamente como sendo o arquivo de um projeto (pom.xml).

Essa é a nossa API, a “forum”. 

Então, o primeiro passo foi feito com êxito, agora vamos para o segundo passo.

Vou abrir o meu Terminal. 

Isso é agnóstico, você pode usar em qualquer sistema operacional. 

As dependências, você vai cumprir e instalar em qualquer sistema operacional, seja Windows, Linux ou MacOS.

Vou entrar onde a aplicação está descompactada, onde está toda a stack. 

Aqui, você vai encontrar o diretório mysql, o docker-compose e o app. 

Vamos começar olhando para o conteúdo que está dentro de mysql.

database.sql
```
CREATE TABLE `forum`.`usuario`(`id` int(11) NOT NULL AUTO_INCREMENT,
	`nome` TEXT NOT NULL,
	`email` VARCHAR(255) NOT NULL,
	`senha` VARCHAR(255) NOT NULL,
	PRIMARY KEY (`id`));
CREATE TABLE `forum`.`perfil`(`id` int(11) NOT NULL AUTO_INCREMENT,
	`nome` TEXT NOT NULL,
	PRIMARY KEY (`id`));
CREATE TABLE `forum`.`usuario_perfis`(`usuario_id` int(11) NOT NULL,
	`perfis_id` int(11) NOT NULL);
CREATE TABLE `forum`.`curso`(`id` int(11) NOT NULL AUTO_INCREMENT,
	`nome` text NOT NULL,
        `categoria` text NOT NULL,
	PRIMARY KEY (`id`));
CREATE TABLE `forum`.`resposta`(`id` int(11) NOT NULL AUTO_INCREMENT,
        `autor_id` int(11) NOT NULL,
	`topico_id` int(11) NOT NULL,
        `mensagem` text NOT NULL,
        `solucao`text NOT NULL,
        `data_criacao` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (`id`));
CREATE TABLE `forum`.`topico`(`id` int(11) NOT NULL AUTO_INCREMENT,
	`titulo` text NOT NULL,
	`mensagem` text NOT NULL,
	`data_criacao` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
	`status` text NOT NULL,
	`autor_id` int(11) NOT NULL,
	`curso_id` int(11) NOT NULL,
        PRIMARY KEY (`id`));
INSERT INTO usuario(nome, email, senha)VALUES('Aluno', 'aluno@email.com', '$2a$10$sFKmbxbG4ryhwPNx/l3pgOJSt.fW1z6YcUnuE2X8APA/Z3NI/oSpq');
INSERT INTO usuario(nome, email, senha)VALUES('Moderador', 'moderador@email.com', '$2a$10$sFKmbxbG4ryhwPNx/l3pgOJSt.fW1z6YcUnuE2X8APA/Z3NI/oSpq');
INSERT INTO perfil(id, nome)VALUES(1, 'ROLE_ALUNO');
INSERT INTO perfil(id, nome)VALUES(2, 'ROLE_MODERADOR');
INSERT INTO usuario_perfis(usuario_id, perfis_id)VALUES(1, 1);
INSERT INTO usuario_perfis(usuario_id, perfis_id)VALUES(2, 2);
INSERT INTO curso(nome, categoria)VALUES('Spring Boot', 'Programacao');
INSERT INTO curso(nome, categoria)VALUES('HTML 5', 'Front-end');
INSERT INTO topico(titulo, mensagem, data_criacao, status, autor_id, curso_id)VALUES('Duvida 1', 'Erro ao criar projeto', '2019-05-05 18:00:00', 'NAO_RESPONDIDO', 1, 1);
INSERT INTO topico(titulo, mensagem, data_criacao, status, autor_id, curso_id)VALUES('Duvida 2', 'Projeto nao compila', '2019-05-05 19:00:00', 'NAO_RESPONDIDO', 1, 1);
INSERT INTO topico(titulo, mensagem, data_criacao, status, autor_id, curso_id)VALUES('Duvida 3', 'Tag HTML', '2019-05-05 20:00:00', 'NAO_RESPONDIDO', 1, 2);
GRANT ALL PRIVILEGES ON *.* TO 'forum'@'%' IDENTIFIED BY 'Bk55yc1u0eiqga6e';
FLUSH PRIVILEGES;

```

É um script SQL e esse script vai ser executado quando o contêiner do MySQL subir para popular a base.

Basicamente, essa é a funcionalidade dele. 

Agora vamos dar uma olhada no docker-compose. É esse cara que vai subir a stack que vamos utilizar.

docker-compose.yaml
```
version: '3'

networks:
  local:

services:
  redis-forum-api:
    image: redis
    container_name: redis-forum-api
    restart: unless-stopped
    ports:
      - 6379:6379
    networks:
      - local

  mysql-forum-api:
    image: mysql:5.7
    container_name: mysql-forum-api
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: 'forum'
      MYSQL_USER: 'forum'
      MYSQL_PASSWORD: 'Bk55yc1u0eiqga6e'
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
      MYSQL_ROOT_HOST: '%'
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d
    ports:
      - 3306:3306
    networks:
      - local
    depends_on:
      - redis-forum-api

```

Eu tenho dois serviços aqui, o redis-forum-api e o mysql-forum-api. 

Esses dois contêineres vão fazer bind de porta com a sua máquina. 

Se você tiver uma instância local rodando MySQL, vai gerar conflito.

Se você tiver o serviço MySQL configurado, derruba ele. 

Se for qualquer serviço ocupando a porta 3306 TCP, você deve derrubar esse serviço para que esse bind ocorra sem gerar nenhum problema.

A mesma coisa para o Redis, que vai rodar na 6379, a porta padrão do Redis fazendo bind para a 6379 da sua máquina. 

Ele vai estar em uma rede local do Docker Compose, em uma rede do Docker, mas ele vai "bindar" na sua porta.

Aqui tem as configurações do MySQL e a dependência dele. 

```
mysql-forum-api:
    image: mysql:5.7
    container_name: mysql-forum-api
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: 'forum'
      MYSQL_USER: 'forum'
      MYSQL_PASSWORD: 'Bk55yc1u0eiqga6e'
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
      MYSQL_ROOT_HOST: '%'
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d
    ports:
      - 3306:3306
    networks:
      - local
    depends_on:
      - redis-forum-api
```

Ele precisa que o Redis suba primeiro para que ele possa subir depois. 

Essas são as dependências básicas de comunicação dessa API

Tendo feito isso, eu posso subir a stack.

digitar este comando na pasta onde esta o arquivo docker-compose.yaml:

```
docker-compose up 
```

Eu não vou subir em daemon agora, eu vou subir prendendo o meu terminal para que você possa acompanhar a subida.

Para você, isso vai demorar um pouco mais. 

Precisamos do serviço do docker rodando:

Então, vamos lá, sudo systemctl start docker docker.socket. 

No Windows, o processo é outro, ele tem, inclusive, um cliente de interface gráfica que facilita a sua vida. Você vai lá e dá um start no serviço.

agora é só aguardar o serviço subir para subir a stack. 

Vou dar uma olhada no status do serviço, com sudo systemctl status docker docker.socket.

Verificar se subiu com êxito. 

Está aí, “active running”, subiu tudo e agora posso dar o meu docker-compose up. 

Ele vai criar as redes, vai criar os contêineres.

Com isso, temos a base necessária para poder rodar a nossa aplicação.

Eu vou abrir o meu Terminal e vou entrar em app/, com cd app/. Em app/, vou rodar o comando mvn clean package.

É o Maven, estou chamando o Maven para fazer um build da aplicação e verificar se está tudo certo com esse pacote que eu baixei. 

Ele vai executar uma série de testes da aplicação e, se tudo passar, se estiver tudo certo, ele vai concluir esse build e vai gerar um artefato JAR. Passou, ele rodou quatro testes, nenhuma falha, nenhum erro, passou tudo, "buildou" com sucesso.

Tendo feito isso, vou no Eclipse e tenho um script aqui chamado start.sh. 

Basicamente, é só executar esse script para que a aplicação possa subir. 

ou somente executar o projeto como java application.

Quando chegarmos no status final da aplicação com ela instrumentada, com o application properties devidamente configurado, com tudo certo, ela vai virar um contêiner e vai subir diretamente via Docker Compose.

Eu subi a aplicação, ela está aqui. 

Agora, vou no Firefox e vou em “localhost:8080/topicos”. 

Está aqui, está rodando, essa é a nossa API.

Ela responde por “tópicos” e também aceita um ID específico em “tópicos”. 

Lá na base, aquele script SQL populou três IDs lá dentro. 

Tenho “1”, “2” e “3”, são endpoints.

Para cada inserção que ocorrer de tópico dessa aplicação, vai acabar gerando um endpoint com um número daquele ID que você consegue requisitar.

Então, está aqui, configuramos a stack.

Aí entra o nosso problema. 

Se olharmos para essa aplicação hoje, ela não está instrumentada, ela não tem observabilidade. 

Então, eu não sei como está o funcionamento dela de fato.

Ela está respondendo, mas quanto ela está consumindo de CPU? 

Qual o tempo de resposta que os meus clientes estão tendo? 

Quanto ela está usando de memória? 

Como está a conexão com a base de dados? 

Qual a duração de uma requisição? 

Estou tendo algum erro? 

Tem algum cliente com erro?

Tudo isso nós não sabemos, nós estamos cegos. 

Então, vamos fazer a primeira parte da camada de observabilidade para sairmos do escuro e entendermos o funcionamento da aplicação, iniciando pelo Actuator.

## Externalizando métricas com Actuator

Agora, vamos configurar o Actuator e enfim tornar a nossa aplicação observável – ou, pelo menos, iniciar esse trajeto.

nós conseguimos implantar o ambiente e subir a aplicação. 

A primeira coisa que eu peço a você é que você dê uma olhada na documentação do Actuator. 

É só você digitar no Google “Actuator” e você vai encontrar em um dos primeiros resultados a documentação do Spring dizendo como você vai habilitar o Actuator.

[documentaçao do actuator](https://docs.spring.io/spring-boot/docs/3.0.x/actuator-api/htmlsingle/)

[Baeldung](https://www.baeldung.com/spring-boot-actuators)

O Actuator entra como uma dependência. 

No nosso caso, estamos utilizando o Maven. 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Se estiver usando Gradle, é outra forma de implantação, mas, no nosso caso, é o Maven.

Tendo feito isso, é necessário que façamos alguns ajustes na nossa aplicação. 

Nesse caso, não vamos mexer no código da aplicação agora. 

No painel à esquerda, vamos em src/main/resources, vamos no application-prod.properties e aqui tem diversas configurações.

```
server.port:8080

# Redis cache config 
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379

# datasource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/forum
spring.datasource.username=forum
spring.datasource.password=Bk55yc1u0eiqga6e

# jpa
spring.jpa.database = MYSQL
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.format_sql=true

# jwt
forum.jwt.secret=rm'!@N=Ke!~p8VTA2ZRK~nMDQX5Uvm!m'D&]{@Vr?G;2?XhbC:Qa#9#eMLN\}x3?JR3.2zr~v)gYF^8\:8>:XfB:Ww75N/emt9Yj[bQMNCWwW\J?N,nvH.<2\.r~w]*e~vgak)X"v8H`MH/7"2E`,^k@n<vE-wD3g9JWPy;CrY*.Kd2_D])=><D?YhBaSua5hW%{2]_FVXzb9`8FH^b[X3jzVER&:jw2<=c38=>L/zBq`}C6tT*cCSVC^c]-L}&/
forum.jwt.expiration=86400000

info.app.name=@project.name@
info.app.description=@project.description@
info.app.version=@project.version@
info.app.encoding=@project.build.sourceEncoding@
info.app.java.version=@java.version@

```

Tem a porta em que a aplicação sobre, tem a configuração do Redis, tem a configuração do MySQL, tem o JPA para conexão e tem o token jwt, porque essa aplicação usa um token.

Para algumas ações de excluir um tópico, criar um tópico, é necessário ter um token que é gerado através de uma autenticação. Estamos falando de uma API Rest.

O que vou fazer agora? 

Em qualquer espaço que você encontrar, pode colocar actuator. 

```
# actuator
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*
```

A primeira linha que eu inseri aqui foi o management.endpoint.health.show-details.

Está como always, então vai mostrar todos os detalhes, e aqui eu tenho o web.exposure. 

Aqui está um asterisco; se eu deixar com asterisco, ele vai basicamente colocar todas as informações relacionadas a JVM e tudo que estiver relacionado à aplicação, ao gerenciamento da aplicação pela JVM.

Quem eu vou disponibilizar é um endpoint chamado health, um endpoint chamado info e um endpoint chamado metrics. 

```
# actuator
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=health,info,metrics
```

Então: management.endpoints.web.exposure.include=health,info,metrics.

Esses três estão de bom tamanho, são o que nós precisamos. 

O nosso foco vai estar sobre o metrics, mas o health é legal se você tem um endpoint que pode ser usado para health check, dependendo de onde você estiver rodando a sua aplicação.

E o info é legal porque você consegue ter as informações dessa aplicação, informações interessantes para você, não para o público externo. 

**health, info e metrics são internos, não devem ser expostos publicamente.**

Essa aplicação, quando estiver rodando na stack do Docker Compose, vai estar com esses endpoints fechados. 

Para acessarmos essa aplicação, vamos ter que passar por um proxy que também vai ser implantado no próximo capítulo.

Tendo cumprido esses pré-requisitos, eu vou para o Terminal. 

vou rodar um mvn clean package para "rebuildar" essa aplicação.

Nos contêineres, não é necessário mexer. 

vou no start.sh novamente e vou rodar a aplicação.

Se você olhar, essa linha da inicialização, só para você ter uma ideia, de repente você está no Windows e o bin/bash não vai funcionar para você.

Você pode rodar 

```
java -jar -Dspring.profiles.active=prod target/forum.jar

```

na linha de comando do cmd do próprio Windows, contanto que você esteja posicionado no path em que está o app. 

Você tem que estar dentro do diretório app, aí você pode chamar o Java dessa maneira que ele vai encontrar, dentro de target, o forum.

Você só vai ter que fazer isso no Windows, colocar uma contrabarra (target\forum) e não a barra (/), porque o Windows não entende a barra que, no Linux, faz distinção entre um diretório e outro. No MacOS, isso não muda nada.

Voltando, a aplicação subiu, foi "startada" e, por fim, eu tenho “topicos”, tenho “http://localhost:8080/topicos/” e um ID qualquer. 

Agora tenho “http://localhst:8080/actuator”.

```
{
"_links": {
"self": {
"href": "http://localhost:8080/actuator",
"templated": false
},
"health": {
"href": "http://localhost:8080/actuator/health",
"templated": false
},
"health-path": {
"href": "http://localhost:8080/actuator/health/{*path}",
"templated": true
},
"info": {
"href": "http://localhost:8080/actuator/info",
"templated": false
},
"metrics-requiredMetricName": {
"href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
"templated": true
},
"metrics": {
"href": "http://localhost:8080/actuator/metrics",
"templated": false
}
}
}
```

Aqui está o Actuator expondo para nós health. 

Aqui tenho health mais um path específico, mas ele é herdado desse health aqui que foi configurado no application properties.

tenho o endpoint health-path que veio herdado do health, tenho o info, e tenho o metrics com uma métrica específica, o metric name, e tenho o metrics puro.

Então, o que vou fazer aqui? 

Vou abrir o health, vamos dar uma olhada no health. 

```
{
"status": "UP",
"components": {
"db": {
"status": "UP",
"details": {
"database": "MySQL",
"validationQuery": "isValid()"
}
},
"diskSpace": {
"status": "UP",
"details": {
"total": 419429347328,
"free": 277309042688,
"threshold": 10485760,
"exists": true
}
},
"ping": {
"status": "UP"
},
"redis": {
"status": "UP",
"details": {
"version": "7.0.3"
}
}
}
}
```

Está aqui, esse é o health que pode ser usado como health check, então status: UP. 

Ele olha, inclusive, a comunicação que a aplicação depende, no caso, o My SQL, está conectado.

Olha o espaço em disco. Tem o redis também UP. Então, esse health é importantíssimo para a questão de health check e para você entender, em um momento imediato, se alguma dependência está com problema.

Se ele não conectar com o Redis, vai bater aqui; se não conectar com o MySQL, vai bater aqui também, porque isso não depende da aplicação conectar no banco para ser exibido.

Já demos uma olhada no health, vamos dar uma olhada no info, o que eu encontro dentro do info. 

```
{
"app": {
"name": "forum",
"description": "Demo project for Spring Boot",
"version": "0.0.1-SNAPSHOT",
"encoding": "UTF-8",
"java": {
"version": "1.8.0_161"
}
}
}
```

O app, qual é o nome, a descrição, a versão, como ele está codificado e a versão do Java.

O que nós conseguimos, na parte mais importante, em metrics, o que conseguimos ver? 

```
{
"names": [
"hikaricp.connections",
"hikaricp.connections.acquire",
"hikaricp.connections.active",
"hikaricp.connections.creation",
"hikaricp.connections.idle",
"hikaricp.connections.max",
"hikaricp.connections.min",
"hikaricp.connections.pending",
"hikaricp.connections.timeout",
"hikaricp.connections.usage",
"http.server.requests",
"jdbc.connections.active",
"jdbc.connections.idle",
"jdbc.connections.max",
"jdbc.connections.min",
"jvm.buffer.count",
"jvm.buffer.memory.used",
"jvm.buffer.total.capacity",
"jvm.classes.loaded",
"jvm.classes.unloaded",
"jvm.gc.live.data.size",
"jvm.gc.max.data.size",
"jvm.gc.memory.allocated",
"jvm.gc.memory.promoted",
"jvm.gc.pause",
"jvm.memory.committed",
"jvm.memory.max",
"jvm.memory.used",
"jvm.threads.daemon",
"jvm.threads.live",
"jvm.threads.peak",
"jvm.threads.states",
"logback.events",
"process.cpu.usage",
"process.start.time",
"process.uptime",
"system.cpu.count",
"system.cpu.usage",
"tomcat.sessions.active.current",
"tomcat.sessions.active.max",
"tomcat.sessions.alive.max",
"tomcat.sessions.created",
"tomcat.sessions.expired",
"tomcat.sessions.rejected"
]
}
```

**Aqui tenho a exposição de todas as métricas da JVM. **

Se você verificar, você tem o uso de CPU, você tem o log no logback, você tem a questão de threads, de memória, você consegue encontrar o garbage collector.

Você consegue pegar o pool de conexões da JDBC com o banco, conexões pendentes, com timeout, utilizadas, tempo de criação, enfim, diversas métricas.

Essas métricas, se eu pegar o nome, eu consigo chegar nesse endpoint com o nome de uma métrica naquela métrica específica. 

Por exemplo, vou pegar o hikaricp.connections, vou chamar esse endpoint, que é o mesmo. 

Vou colocar o “http://localhost:8080/actuator/metrics/hikaricp.connections” e está aqui, o valor é 10.

**Nós conseguimos colocar as métricas da JVM aqui, elas estão expostas, porém, não estão no formato esperado. **

Nós não conseguimos identificar e usar essas métricas através do Prometheus, que é o nosso objetivo.

Então, vamos configurar o Micrometer e ele vai fazer esse meio de campo, vai tornar essas métricas legíveis para o Prometheus.

## Expondo métricas para o Prometheus

fizemos a configuração do Actuator e conseguimos externalizar as métricas da JVM.

Essas métricas ajudam bastante, mas não são o que nós realmente queremos, nós precisamos que as métricas estejam legíveis para o Prometheus. 

**Para conseguirmos fazer isso, temos que trabalhar com o Micrometer.**

Basicamente, ele vai ser a nossa fachada de métricas, levando as métricas em um formato legível para o Prometheus.

Para fazer isso, é bem simples, basta você dar uma pesquisada em “Micrometer”. 

[micrometer](https://micrometer.io/)

Entrando no site, vou em “Documentação > Instalação”. 

Na “Instalação”, eu tenho o formato para o Gradle – não é o que nos atende, eu vou utilizar o bloco para o Maven.

```
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Você vai copiar esse bloco e, de posse desse conteúdo copiado, vamos no pom.xml adicionar essa dependência. 

Basta salvar e agora vem uma alteração no application.prod.properties.

O que vai ser necessário? 

Colocar o endpoint Prometheus, que nós não colocamos na última aula. 

Logo após metrics, vai colocar uma vírgula e vai colocar aqui prometheus: management.endpoints.web.exposure.include=health,info,metrics,prometheus.

```
# actuator
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=health,info,metrics, prometheus
```

Vamos expor o endpoint prometheus e, além disso, precisamos de uma configuração adicional. 

```
# prometheus
management.metrics.enable.jvm=true
management.metrics.export.prometheus.enabled=true
management.metrics.distribution.sla.http.server.requests=50ms,100ms,200ms,300ms,400ms,500ms,1s
management.metrics.tags.application=app-forum-api
```

é muito importante que vocês entendam cada uma dessas linhas.


Primeiro, vamos habilitar as métricas da JVM, tem aqui management.
metrics.enable.jvm=true; 

tem aqui também o management.metrics.export.prometheus.enabled=true. 

Estamos habilitando o export das métricas para o Prometheus.

Temos aqui uma métrica específica relacionada à SLA, já vamos deixar configurado para não termos que voltar nesse arquivo no futuro.

Aqui também tenho uma configuração que é de tagueamento. 

**Quando externalizarmos uma métrica, essa métrica vai ser exibida com um tagueamento. **

Nesse caso, ela vai ter a tag **app-forum-api**.

Por que isso? 

Quando tivermos N aplicações rodando, algumas métricas vão ser iguais.

Por exemplo, a métrica de tempo de resposta pode ter o mesmo nome, mas posso querer olhar mais de uma aplicação, aí a tag entra nesse ponto, porque eu posso selecionar aquela métrica com uma tag específica correspondente à aplicação que eu quero avaliar.

Vamos para o Terminal e eu vou fazer o build dessa nova versão.

Enquanto ele vai "buildando", podemos dar uma olhada no Actuator. 

Notem que são esses endpoints que nós possuímos.

Não tem nada relacionado ao Prometheus – temos métricas, mas não temos o Prometheus. 

Vamos no start.sh e vamos rodar a aplicação.

Uma coisa importante, que eu gostaria que vocês fizessem, é dar uma lida na documentação do Actuator. 

Aqui, no Actuator, você pode ver a parte de “Logging”, que é exposta, e tem o log level, que é exposto.

Tem a parte de métricas. 

Já veio o Micrometer aqui e tem a parte específica do Prometheus. 

```
"prometheus": {
"href": "http://localhost:8080/actuator/prometheus",
"templated": false
},
```

E também dar uma olhada na documentação do Micrometer.

É bem importante que você entenda isso para que você possa levar para outras aplicações com o conhecimento de causa interessante. 

vamos voltar para o browser. 

No browser, vou atualizar o Actuator e agora já tem o endpoint actuator-prometheus. 

http://localhost:8080/actuator/prometheus

Clicando nele, nós encontramos as métricas no formato esperado pelo Prometheus.

Se você notar, tem bastante coisa, é muita métrica, e a maior parte delas vai ser útil para nós na nossa implantação, no acompanhamento do funcionamento da nossa API. 

Alguma aqui não é útil? 

Na verdade, nenhuma é inútil, todas elas são úteis.

Algumas vamos acabar desprezando quando estamos rodando a nossa aplicação em um contêiner, então não são tão necessárias para nós quanto o número de núcleos de uma CPU ou coisas do tipo.

Mas está aqui, conseguimos expor essas métricas em um formato legível para o Prometheus e agora podemos entender o que são essas métricas e ao que cada uma delas corresponde.

Pelo menos, as que são interessantes para a nossa aplicação e que vamos utilizar futuramente configurando dashboards e alertas em cima de valores específicos, de momentos específicos dessas métricas.

## Métricas da JVM

Essas métricas estão sendo externalizadas através do Micrometer e o endpoint em que vamos consumir essas métricas ficam em 

http://localhost:8080/actuator/prometheus

Isso é só por enquanto. 

Lá na frente, vamos acessar isso através de outro endpoint porque vamos passar por um proxy. 

O Prometheus não vai passar pelo proxy, ele vai de forma interna, mas quando formos visualizar, a visualização será outra.

No momento, vamos manter assim. 

Vamos dar uma olhada para entender quais informações nós conseguimos coletar. 

### Memória
Temos aqui a parte de utilização de memória com essa métrica **jvm_memory_used_bytes**. 

É muito legal porque você consegue verificar como está a alocação de memória heap – o que está em área heap e o que está em área nonheap.

Além disso, não vou falar de todas as métricas, vou falar só das mais importantes para nós e de outras que podem ser interessantes para você, mas não de todas.

### Logs
Aqui, temos uma métrica **logback_events_total**. 

Essa métrica, basicamente, está relacionada àquele tipo de evento registrado no log. 

Você consegue verificar info, trace, warn, error e debug. 

é uma métrica importante.

### Hikaricp -- Conexoes e Banco de dados
Além disso, temos aqui o hikaricp que é o monitoramento das conexões da aplicação com a base de dados.

temos a **hikaricp_connections_min**, o número mínimo, essa métrica está relacionada ao número mínimo de conexões, que é o pool de conexões que é aberto quando a aplicação sobe.

**hikaricp_connections_usage_seconds_max**
Aqui são os segundos utilizados para fazer a conexão. 

### Process
Andando um pouco mais sobre essas métricas, está aqui **process_files_open_files**, 66; 

### JVM
**jvm_gc_pause_seconds_...**
uma métrica bem legal para você entender como o garbage collector está fazendo o trabalho dele, o tempo de uma execução entre outra do garbage collector.

### CPU
Além disso, temos aqui **process_cpu_usage**, temos a utilização de CPU por conta do processo que está em execução pela aplicação. 

### JVM Memory
**jvm_memory_commitetted_bytes**
temos mais uma métrica relacionada à memória. 

Aqui, no caso, é a alocação nas áreas de memória pela JVM.

### hikarycp -- tempo para aquisicao de conexao
**hikaricp_connections_acquire_seconds**

temos o tempo que demorou para que a conexão fosse iniciada com a base de dados; 

### JVM Threads
**jvm_threads_states_threads**

temos também métricas relacionadas ao estado das threads – isso é bem legal, você consegue ver quais estão em execução, quais estão em espera, quais foram terminadas, quais estão em time-waiting, blocked e new.

### SLA
Lembra da métrica de SLA que nós colocamos – que, na verdade, nós configuramos via application properties? 

Na verdade, o que nós configuramos foi a exibição da métrica.

Você vai notar que é o **http_server_requests_seconds_bucket**. 

Existe um bucket que vai alocar esses valores para uma determinada contagem. 

Temos aqui de 50 milissegundos, 100, 200, 300, 500, 1 segundo e ao infinito e além. 

**Passou de 500 milissegundos, temos um problema; de 1 segundo em diante a coisa está bem crítica em termos de resposta para a nossa API.**

Está aqui uma métrica que é importantíssima para entendermos como está o desempenho da nossa aplicação e a experiência do usuário final ao consumi-la, porque você não vai querer que a sua API demore mais de um segundo para responder.

De 300 milissegundos para cima, já um caso a se preocupar. 

Se, porventura, algo ficar indisponível e tivermos um 500 aqui, vai "plotar" essa métrica, ela vai ser exibida relacionada ao status de erro 500.

### JVM classes
jvm_classes_loaded_classes

### timeout de conexao com o database
**hikarycp_connections_timeout_total**

aqui o timeout de conexão com o database, uma métrica bem importante para entendermos, no momento de um incidente, se a nossa aplicação não está se comunicando com a base de dados e se isso foi o causador do nosso problema.

### tempo de criaçao de uma conexao com o database
**hikaricp_connections_creation_seconds**
O tempo de criação de uma conexão, esse é o tempo máximo, no caso. 


### utilizaçao de CPU
**system_cpu_usage**
Utilização de CPU, está aqui a utilização de CPU. 

tem a de CPU, tem a de processo. 

Essa daqui, **system_cpu_count**, não vai ser tão interessante quando nós estivermos rodando a nossa aplicação em um modelo distribuído porque, basicamente, ela traz a contagem de núcleos de CPU, mas está valendo, é uma métrica também.

Eu não vou me alongar muito nesse assunto, a documentação dessas métricas está naqueles links que eu passei nas aulas anteriores que contemplam o Actuator e o Prometheus.

Mas é interessante que saibamos a riqueza das informações que são exibidas aqui para que possamos realmente observar o que é importante para o nosso caso.

Pode ser que você tenha um caso diferente desse que nós vamos enfrentar nessa aplicação, então pode ser interessante para você olhar para outras métricas no futuro e você já sabe onde encontrá-las e como aprofundar no que realmente são essas métricas através da documentação.

Então, vamos entender como é o processo de instrumentação e de utilização de uma métrica personalizada.

## Métricas personalizadas
O problema é o seguinte: temos basicamente todas as métricas da JVM sendo expostas, porém, nós não temos nenhuma métrica que corresponda a alguma regra de negócio da nossa aplicação, e temos alguns processos que precisam ser mapeados.

Hoje, não temos o controle sobre o número de usuários autenticados e as tentativas de autenticação com erro. 

Vamos trabalhar em cima disso e gerar uma métrica para autenticação com sucesso e outra para erros de autenticação.

Vamos abrir a IDE, vamos para o Eclipse. 

No Eclipse, vamos na nossa aplicação, expandir o main/java e vamos abrir o pacote forum.controller.

Vamos trabalhar no controller. 

Dentro do controller, temos uma classe que é a AutenticacaoController.java que faz o controle de autenticação. 

```
@RestController
@RequestMapping("/auth")
@Profile(value = {"prod", "test"})
public class AutenticacaoController {
	
	@Autowired
	private AuthenticationManager authManager;
	
	@Autowired
	private TokenService tokenService;
	
	@PostMapping
	public ResponseEntity<TokenDto> autenticar(@RequestBody @Valid LoginForm form) {
		UsernamePasswordAuthenticationToken dadosLogin = form.converter();
		
		try {
			Authentication authentication = authManager.authenticate(dadosLogin);
			String token = tokenService.gerarToken(authentication); 		
			return ResponseEntity.ok(new TokenDto(token, "Bearer"));
			
		} catch (AuthenticationException e) {
			return ResponseEntity.badRequest().build();
		}

		
	}
}

```

Basicamente, o que essa classe faz – vou explicar de forma bem resumida – é, se você passa um usuário e uma senha correta, ela vai te retornar um token; caso contrário, ela te retorna um erro já tratado.

Esse é o básico dessa classe para não termos que mergulhar em cima da lógica feita nessa aplicação. 

Então, o que precisamos aqui? 

Primeiro, precisamos de umas dependências, mas essas dependências vão ser cumpridas conforme formos criando o método que vamos utilizar.

Eu vou começar definindo dois atributos. 

Eles vão se chamar **authUserSuccess** e **authUserErrors**. 

Ao inserir esses dois atributos, você vai ver que o Eclipse mostra um problema.

O que é esse problema? Temos que importar uma biblioteca, temos que importar uma dependência para que isso funcione. Presta bem atenção aqui porque eu vou trabalhar com o Counter vindo do **io.micrometer.core.instrument**.

Importando isso não temos mais aquela sinalização de erro. 

Agora, vou criar o método aqui. 


```
public AutenticacaoController(MeterRegistry registry) {
		authUserSuccess = Counter.builder("auth_user_success")
				.description("usuarios autenticados")
				.register(registry);
		
		authUserError = Counter.builder("auth_user_error")
				.description("erros de login")
				.register(registry);
	}
```

note que também está pedindo mais uma dependência. Eu sempre vou trabalhar com o io.micrometer.core.instrument.

agora vamos entender isso. 

Eu tenho um método, os parâmetros que ele necessita é o MeterRegistry e o registry. 

Aqui, eu tenho uma propriedade que é o authUserSuccess, que eu já instanciei aqui em cima, o atributo.

Basicamente, o que ela faz? 

Ela cria uma métrica chamada **auth_user_success** com uma descrição ”usuarios autenticados” e anexa isso no registry, então ela registra essa métrica para mim.

Aqui, eu tenho também o meu atributo **authUserErrors** e ele faz exatamente a mesma coisa, Counter.Builder, cria o auth_user_error, cria a descrição ”erros de login” e faz o registro dessa métrica.

Tendo feito isso, já estamos trabalhando com uma métrica que é do tipo Counter. 

**Uma métrica do tipo Counter significa que é uma métrica incremental, ela vai crescer gradativamente, conforme um evento específico for disparado.**

Que evento é esse que vai trabalhar com o crescimento dessa métrica, que vai alimentar a configuração dela? 

Se olharmos aqui, temos o ResponseEntity que retorna um TokenDto.

Esse token é o token de autenticação. 

Estamos falando de uma API Rest, ela não guarda o estado e o processo de autenticação corresponde à requisição de um front-end ou de um app que vai trazer um usuário e uma senha que vão chegar nessa API.

Ela vai encaminhar essa requisição para a lógica de validação que ela possui, consultar o banco, e, se estiver tudo certo, ela retorna um token para esse usuário continuar executando suas ações autenticado através da API. Então, essa é a lógica.

Aqui, temos um bloco try/catch. Dentro desse bloco, eu tenho, logicamente no try, a minha primeira opção é a de sucesso. Vou basicamente trabalhar com o incremento desse valor.

No meu bloco try, vou incrementar essa chamada authUserSuccess.increment(). 

```
try {
			Authentication authentication = authManager.authenticate(dadosLogin);
			String token = tokenService.gerarToken(authentication); 
			authUserSuccess.increment();
			return ResponseEntity.ok(new TokenDto(token, "Bearer"));
			
		} 
```

Bem simples, a mesma coisa vou fazer com Errors authUserErrors.increment().

```
catch (AuthenticationException e) {
			authUserErrors.increment();
			return ResponseEntity.badRequest().build();
		}
```

Tendo feito essa configuração, salva o código e já podemos fazer o build desse novo artefato. 

vamos subir a aplicação. 

Vamos voltar para o browser, vamos atualizar e vamos procurar por user agora. 

Aqui, auth_user_success_total usuarios autenticados, é uma métrica do tipo counter, e está aqui, não tem nenhum usuário autenticado. 

Vamos para o error.

Está aqui, auth_user_error_total erros de login, também do tipo counter, e está aqui a métrica auth_user_error 0.0. Nenhum erro e nenhum usuário autenticado.

Com o Postman aberto, você vai fazer uma requisição. 

Essa requisição é “Post”, você define como “Post”, vamos fazer para “localhost:8080/auth”. 

Em “Authorization”, entra um “Bearer Token”, então o “Type” é “Bearer Token”. 

Em “Header”, é importante você colocar o “Content Type”, que é “application/json”, senão não vai funcionar. E no “Body” você vai colocar esse JSON aqui:

```
{
    "email": "moderador@email.com",
    "senha": "123456"
}
```

Isso é o que você precisa configurar no Postman para fazer essa requisição. 

O retorno da API para mim é esse, ele me retorna o token do tipo "Bearer". 

Então, eu tenho agora um usuário autenticado. 

Então, vou atualizar e está aqui, agora já temos um 1 usuário autenticado.

Não está fazendo distinção se é o usuário X ou Y, é só o número de autenticações válidas. 

Tendo feito isso, vou colocar um caractere a mais para essa autenticação resultar em um erro:

```
{
    "email": "moderador@email.com",
    "senha": "1234567"
}
```

Vamos voltar nas nossas métricas, vamos olhar essa outra métrica de usuário. Aqui é o total de usuários autenticados e aqui estão os erros. 

Vou atualizar, agora estou com 16 erros e vamos procurar o sucesso. 

O sucesso continua com 13.

Nós criamos uma métrica personalizada, agora podemos mensurar quantos usuários autenticaram em um período, é isso que vamos fazer quando realmente trabalharmos com essa métrica, porque vamos olhar uma série temporal.

Vamos entender quantos usuários autenticaram no último minuto e quantos erros de autenticação ocorreram também no último minuto ou nos últimos N minutos.

Eu recomendo que você leia com atenção, que você estude essa documentação para você implementar outras métricas nessa aplicação e, futuramente, levar isso para uma aplicação sua.

## Preparando a API para a conteinerização

Vamos preparar essa aplicação para ela ser empacotada e rodar junto com a stack do Docker Compose.

O que vamos fazer é o seguinte, vou fechar o “src/main/java” e vou no “src/main/resources”. 

No application.prod.properties, vou modificar o redis.host. 

Aqui está para localhost, mas esse contêiner se chama redis-forum-api.

disto:

```
spring.redis.host=localhost
```

para isto:

```
spring.redis.host=redis-forum-api
```

Em spring.datasource, vou trocar o localhost pelo mysql-forum-api.

disto:
```
spring.datasource.url=jdbc:mysql://localhost:3306/forum
```
para isto:
```
spring.datasource.url=jdbc:mysql://mysql-forum-api:3306/forum
```

temos que fazer essa mudança porque essa API vai conectar tanto no MySQL quanto no Redis por uma rede interna do Docker. 

Então, o nosso Docker Compose vai mudar.

No proximo topico, você já vai ter um Docker Compose diferente e esse Docker Compose já vai subir uma stack maior, ele vai subir o Redis e o MySQL internamente, o próprio Docker Compose vai fazer o build do contêiner da aplicação e, além disso, ele vai subir também o Prometheus.

Agora, com relação à aplicação, a conteinerização dela é bem simples.

o que precisamos mudar aqui, como pré-requisito, são apenas esses dois valores. 

Podemos fechar a aplicação, não vamos "startar" ela por aqui, vou ver o que está executando da stack.

vou "rebuildar" a aplicaçao. 

Vamos criar um artefato JAR que agora já possui essas configurações, ele já está configurado para conectar no contêiner, não mais no localhost.

A partir de agora, já vamos trabalhar de uma forma diferente, vamos focar nas ferramentas-chaves do curso – nesse caso, o Prometheus, o Grafana e, posteriormente, o Alert Manager.

## Subindo a stack com API e Prometheus
O que vamos fazer? 

Vamos dar uma olhada no arquivo docker-compose. 
```
version: '3'

networks:
  database:
    internal: true
  cache:
    internal: true
  api:
    internal: true
  monit:
  proxy:

services:
  redis-forum-api:
    image: redis
    container_name: redis-forum-api
    restart: unless-stopped
    expose:
      - 6379
    networks:
      - cache

  mysql-forum-api:
    image: mysql:5.7
    container_name: mysql-forum-api
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: 'forum'
      MYSQL_USER: 'forum'
      MYSQL_PASSWORD: 'Bk55yc1u0eiqga6e'
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
      MYSQL_ROOT_HOST: '%'
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d
    expose:
      - 3306
    networks:
      - database
    depends_on:
      - redis-forum-api

  app-forum-api:
    build:
      context: ./app/
      dockerfile: Dockerfile
    image: app-forum-api
    container_name: app-forum-api
    restart: unless-stopped
    networks:
      - api
      - database
      - cache
    depends_on:
      - mysql-forum-api
    healthcheck:
      test: "curl -sS http://app-forum-api:8080/actuator/health"
      interval: 1s
      timeout: 30s
      retries: 60
  
  proxy-forum-api:
    image: nginx
    container_name: proxy-forum-api
    restart: unless-stopped
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/proxy.conf:/etc/nginx/conf.d/proxy.conf
    ports:
      - 80:80
    networks:
      - proxy
      - api
    depends_on:
      - app-forum-api

  prometheus-forum-api:
    image: prom/prometheus:latest
    container_name: prometheus-forum-api
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - monit
      - api
    depends_on:
      - proxy-forum-api
```

Vou abrir o docker_compose.ymal (vim docker-compose.yaml), e vamos falar um pouco desse arquivo antes de subir a stack. 

vamos às alteraçoes.

redes internas do Docker.
```
networks:
  database:
    internal: true
  cache:
    internal: true
  api:
    internal: true
  monit:
  proxy:
```

Agora eu tenho uma rede chamada database, que é interna; tenho uma rede chamada cache, que também é interna; uma rede chamada api, que é interna; e tenho uma rede chamada monit, que, no momento, está externa, porque vamos utilizar a interface web do Prometheus para entender como funciona a linguagem PromQL e compor as nossas primeiras métricas.

Futuramente, ela vai também ser uma rede interna porque Prometheus server não precisa estar exposto. 

E a rede proxy, que é a rede que vai conduzir as requisições à aplicação, requisições externas.

Os clientes vão consumir essa aplicação, mas, antes de chegar na aplicação, eles serão filtrados por um proxy, e é esse proxy que estará exposto, é um nginx.

**O Prometheus vai consumir a aplicação por dentro, então, ele não vai passar pelo proxy, mas os clientes vão passar.**

O que mudou nos serviços? 

```
redis-forum-api:
    image: redis
    container_name: redis-forum-api
    restart: unless-stopped
    expose:
      - 6379
    networks:
      - cache
```

O serviço do Redis está aqui, não mudou basicamente nada, exceto a rede. 

A network dele agora é a cache. 

Ele continua somente com o expose: 6379.

Antes, ele estava fazendo bind de portas, estava com o ports, agora é o expose, ele só expõe a porta do contêiner, a 6379, e isso somente de forma interna, só dentro do Docker.


```
mysql-forum-api:
    image: mysql:5.7
    container_name: mysql-forum-api
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: 'forum'
      MYSQL_USER: 'forum'
      MYSQL_PASSWORD: 'Bk55yc1u0eiqga6e'
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
      MYSQL_ROOT_HOST: '%'
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d
    expose:
      - 3306
    networks:
      - database
    depends_on:
      - redis-forum-api
```

Aqui, temos o mysql-forum-api que também teve mudanças, agora ele não tem mais ports, só tem o expose, então a porta 3306 do MySQL só é visível dentro do Docker e para quem tem acesso à rede database.

Está na rede database e continua com a mesma dependência, que o redis, o redis tem que subir primeiro. 

```
app-forum-api:
    build:
      context: ./app/
      dockerfile: Dockerfile
    image: app-forum-api
    container_name: app-forum-api
    restart: unless-stopped
    networks:
      - api
      - database
      - cache
    depends_on:
      - mysql-forum-api
    healthcheck:
      test: "curl -sS http://app-forum-api:8080/actuator/health"
      interval: 1s
      timeout: 30s
      retries: 60
```

**Agora eu tenho a aplicação que vai ser "buildada", é a nossa app-forum-api.**

Ele vai fazer um build, cujo contexto é o diretório app. 

Ele vai encontrar um dockerfile dentro do diretório app e vai fazer o build, gerando a imagem app-forum-api.

O nome do contêiner vai ser app-forum-api. 

A aplicação está em comunicação direta com três redes: a rede api, a rede database e a rede cache. 

Ela precisa falar com o MySQL e precisa falar com o redis.

Quais são as dependências desse contêiner? 

O MySQL, ele precisa do MySQL para poder consultar a base de dados; e o MySQL, por sua vez, depende do Redis.

Então, tem uma cadeia de dependências que precisa ser cumprida, e ele tem o healthcheck. 

Nós passamos um curl, que vai internamente, com o contêiner app-forum-api:8080/, lá no actuator/health.

Se ele retornar 200, está certo. 

e as configs do timeout e as tentativas. 

```
proxy-forum-api:
    image: nginx
    container_name: proxy-forum-api
    restart: unless-stopped
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/proxy.conf:/etc/nginx/conf.d/proxy.conf
    ports:
      - 80:80
    networks:
      - proxy
      - api
    depends_on:
      - app-forum-api
```

Aqui está o contêiner do proxy, que é um nginx, ele tem diversas rotas que eu vou mostrar para vocês de forma rápida.

Basicamente, esse vai fazer o bind de porta na porta 80 da sua máquina. 

A partir desse momento, as requisições vão ser para localhost, quando quisermos acessar a aplicação de forma externa. 

Ele pertence à rede proxy e api e depende do app-forum-api para subir.


```
prometheus-forum-api:
    image: prom/prometheus:latest
    container_name: prometheus-forum-api
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - monit
      - api
    depends_on:
      - proxy-forum-api
```
Por último, temos o contêiner do Prometheus. 

O nome será prometheus-forum-api; a imagem é o prom/prometheus:latest, então é a última versão, a mais atual; o nome do contêiner é prometheus-forum-api; e aqui os volumes do Prometheus.

Você vai notar que, existem algumas mudanças, ele tem uma pasta chamada prometheus com um subdiretório prometheus_data com o arquivo Prometheus.yml.

Assim como ele tem um diretório nginx que também tem um arquivo chamado nginx.conf e proxy.conf. 

Aqui, estão as configurações do Prometheus para sua subida. 

```
command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
```


Tenho um arquivo de configuração que, se você notar, vai ser retirado do diretório prometheus/prometheus.yml e vai ser colocado no local em que estamos alimentando o config.file como parâmetro.

Esse arquivo tem as configurações do Prometheus. 

Aqui, temos o path do TSDB, do storage dele, que também é um volume que vai sair da nossa máquina.

Temos aqui as bibliotecas que o Prometheus vai utilizar; a porta que ele vai fazer bind, a 9090; e a rede que o Prometheus faz parte que, nesse caso, é a monit e a rede api. 

Está feita a configuração, vou salvar. 

Agora podemos dar um docker-compose up -d, vou subir em modo daemon para não ocupar a tela.

já criou a stack, aparentemente subiu, vamos validar isso para garantir que está tudo certo. 

Agora, qual é a mudança? 

Vou em “http://localhost/topicos”, vamos ver se ele trouxe os tópicos para mim.

O proxy está funcionando, está aqui, “/topicos”. 

Eu posso agora ir em “http://localhost/topicos/2”, trouxe para mim, a aplicação está funcionando. 

Qual é a mudança aqui? Quando quisermos acessar o endpoint de métricas, vamos acessar o “http://localhost/metrics”.

Aqui, nós caímos nas métricas do Prometheus. 

Se quisermos acessar outros endpoints que são expostos pela aplicação através do Actuator, podemos também acessar o “http://localhost/health”, que vai estar aqui e o “http://localhost/info”, que também está disponível para nós.

Onde que está essa configuração? 

Se dermos um ls, você vai ver que eu tenho o nginx, e aqui eu tenho o nginx.conf, que tem a configuração de redirecionamento.

```
user nginx;
worker_processes 1;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/conf.d/proxy.conf;
    include /etc/nginx/mime.types;
    include /etc/nginx/fastcgi_params;
    include /etc/nginx/scgi_params;
    include /etc/nginx/uwsgi_params;
    
    index index.html index.htm;
    
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
  
    access_log off;
    server_tokens off;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;

    server {

        listen 80 default_server;
        listen [::]:80 default_server;
        
        server_name _;
        root /usr/share/nginx/html;

        location /topicos {
            proxy_pass http://app-forum-api:8080/topicos;
        }

        location ~ /topicos/([0-9]+)$ {
            proxy_pass http://app-forum-api:8080/topicos/$1;
        }
        
        location /auth {
            proxy_pass http://app-forum-api:8080/auth;
        }

        location /info {
            proxy_pass http://app-forum-api:8080/actuator/info;
        }
        
        location /metrics {
            proxy_pass http://app-forum-api:8080/actuator/prometheus;
        }
        
        location /health {
            proxy_pass http://app-forum-api:8080/actuator/health;
        }

        location /images {
        access_log off;
        return 204;
        }
    }
}


```

Não precisa se preocupar com isso, é só para você saber onde está. 

Existe essa configuração de proxy_pass que faz o redirecionamento das requisições.

A configuração de proxy não tem nada de interessante para você, pelo menos nesse momento, isso não deve fazer muito sentido, mas é legal você entender como funciona essa configuração.

**Você vai notar que o proxy só serve para acessar a aplicação e os endpoints da aplicação de forma externa.**

Ele não acessa o Prometheus. 

O Prometheus em si está agora em “http://localhost:9090”.

## Conhecendo as configurações do Prometheus
Agora, vamos entender um pouco melhor essa interface web do Prometheus e entender a configuração que está por trás disso.

Para você acessar o Prometheus, é “http://localhost:9090”. 

### Graph
Acessando o Prometheus, você vai cair diretamente aqui, nessa tela, e vai ver que tem aqui o histórico, que pode ser habilitado para consultas; a utilização do tempo local, você pode habilitá-lo ou manter o padrão.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/2.PNG)

Você vê que existem, nesse painel, algumas opções. 

Em evaluation time, você encontra basicamente o tempo em que você vai executar uma consulta, então você pode definir o tempo em que você quer rodar aquela consulta.

Existe o modo tabulado de exibição e o modo gráfico, em que nós vamos acabar vendo um gráfico mesmo. 

Também tem a opção de adicionar painéis.

Vamos entender essas opções de cima. 

Futuramente, não vamos trabalhar com essa interface, porque ela é muito simples para o propósito que nós temos, mas ela é extremamente funcional para você validar uma consulta, para você construir uma métrica, fazer um indicador.

Além de ter alguns outros recursos que vamos utilizar na hora de verificar se temos uma regra de alerta funcional, na hora de verificar se o service discovery está olhando para todas as aplicações que nós configuramos etc.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/3.PNG)

### Alerts
Clicando em “Alerts”, não temos nenhum alerta configurado ainda, isso vamos fazer mais para frente, em outro capítulo, quando trabalharmos o Alertmanager.

### Status
em “Status”, temos algumas informações legais, como runtime e informação de build – não vou aprofundar muito nisso, é só mesmo para você entender o que é cada uma dessas opções.

Em TSDB status está o status do TSBD, que é o Time Series Database. 

Você pode dar uma olhada com mais calma e entender essa configuração. 

No momento, isso não vai fazer diferença para nós.

Em Command-line flags estão as flags de linha de comando que você pode usar para subir o Prometheus na hora da execução dele. Nós usamos algumas naquele Docker Compose.

tem diversas flags que podem ser utilizadas na hora de subir o serviço.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/4.PNG)

tem a “Configuração”. Nessa configuração, vamos olhar o arquivo que gerou essa configuração. Essa configuração está sendo derivada de um arquivo, já vamos olhar para ele, mas é legal você entender que essa configuração você vai poder validar aqui.

Você vem em “Status > Configuration” e você vai ver essa configuração. 

Em “Regras”, não tem nenhuma regra habilitada, não temos o que exibir.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/5.PNG)

Quando você vem em “Targets”, você pode ver quais são os targets que o Prometheus está olhando, quais são os alvos de coleta. 

Temos aqui o contêiner “app-forum-api-8080/actuator/prometheus”, esse é o endpoint que o Micrometer disponibilizou para nós com as métricas traduzidas em um formato legível para o Prometheus.

Aqui, temos um endpoint que é o próprio Prometheus, ele olha para ele mesmo. 

O Prometheus monitora ele mesmo e "plota" essas métricas. 

Se colocarmos “localhost:9090” – o Prometheus roda na porta “9090” TCP – e colocarmos um “localhost:9090/metrics”, vamos pegar as métricas do próprio Prometheus.

Como é uma aplicação feita em Go, você vai ver muitas métricas relacionadas à execução do Go. 

Tem bastante métricas, eu não vou entrar no mérito das métricas do Prometheus, mas é interessante você entender que você pode, em algum momento, ter uma curiosidade do funcionamento do Prometheus, de algum aspecto que parece que não está interessante e olhar as métricas dele.

Se você for em “Unhealthy”, mostra as aplicaçoes que nao estao funcionando corretamente. 

Basicamente, está tudo certo. 

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/6.PNG)

Aqui, o “Service Discovery”. Para onde estamos olhando no momento, é o “app-forum-api” e para o próprio “prometheus-forum-api”.

### Help
temos o “Help”, que nos leva direto para a documentação do Prometheus. 

Agora que já falamos um pouco da interface gráfica do Prometheus – só ressaltar que é aqui que vamos fazer as consultas das nossas métricas. 

vamos focar nessa visualização e, principalmente, em “Graph”.

### Prometheus configuration em desenv
**Aqui no prometheus, tem uma coisa que eu jamais faria em produção, mas, nesse ambiente "conteinerizado", foi feito.**

O diretório prometheus_data está com permissão 777. 

O diretório somente dentro dele não está com 777, mas, se eu olhar só para o diretório, você vai ver que ele está como 777.

Por quê? Porque ele está no path do meu usuário no Linux, e não é o mesmo usuário de execução do Prometheus, que logicamente não é o root. Então, ele não tinha permissão para acessar esse compartilhamento.

Para esse diretório específico, eu liberei um chmod 777 prometheus_data, foi o que eu fiz para que o contêiner funcionasse direito, por isso a coloração estranha no diretório.

**Isso é uma peculiaridade do Linux.** No Windows, provavelmente você vai ter que olhar esse diretório e ir nas propriedades da pasta, “Segurança > Segurança Avançada” e mudar as permissões para outros, permitindo que qualquer usuário que tenha acesso a essa pasta possa acessá-la e escrever nela.

Dificilmente você vai precisar fazer alguma mudança porque estamos falando de uma "conteinerização" em Linux. Enfim, seguindo, além desse diretório, temos o mais importante da aula que é o prometheus.yml.

### Prometheus.yaml
```
global:
  scrape_interval: 5s
scrape_configs:
- job_name: prometheus-forum-api
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - prometheus-forum-api:9090
- job_name: app-forum-api
  metrics_path: /actuator/prometheus
  static_configs:
  - targets:
    - app-forum-api:8080

```

Esse arquivo de configuração gerou aquela configuração que nós vimos lá na interface web. 

Aqui, o que é importante? 

#### scrape interval
scrape_interval, que é uma característica global nessa configuração e em qualquer outra.

O que muda aqui? 

Eu defini como 5s. 

A regra é que, para qualquer configuração de scrape – **scrape é o tempo que o Prometheus demora para fazer uma consulta em um endpoint de métrica**, então, scrape_interval, scrape_time, enfim, é o tempo do Prometheus entre uma consulta e outra em um endpoint de métricas.

Eu coloquei 5s, isso está global. 

Eu posso mudar isso? 

Posso, é só fazer uma configuração diferente em algum job. 

#### Prometheus job
Como é a divisão disso? 

Eu tenho aqui scrape_interval, que é uma característica global nessa configuração, e tenho scrape_configs, que são as configurações de scrape personalizadas.

Se eu quiser sobrescrever isso e mudar o valor em alguma scrape_config, eu tenho que trabalhar dentro do escopo de um job. 

```
- job_name: prometheus-forum-api
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - prometheus-forum-api:9090
```


Tenho aqui job_name: prometheus-forum-api, que é aquela consulta que o Prometheus faz nele mesmo.

Aqui, o meu scrape_interval é de 15s; o timeout de 10s. 

Então, ele está sobrescrevendo a configuração global aqui. 

Qual é o metrics_path que ele olha, qual o path de métricas? É o /metrics.

Qual é o esquema? 

http, o protocolo HTTP. 

As configurações estáticas são o target dele, que é o prometheus-forum-api:9090. 

**Importante, tem que ter o IP, o FQDN da máquina ou o hostname, a porta TCP, onde o Prometheus está rodando**.

Aqui, em metrics_path, o path, o endpoint em que a métrica está. 

É legal que uma informação complementa a outra. 

Aqui, tenho outro job_name que, nesse caso, é o app-forum-api.

```
- job_name: app-forum-api
  metrics_path: /actuator/prometheus
  static_configs:
  - targets:
    - app-forum-api:8080
```

O metrics_path dele é em /actuator/prometheus e a static_configs tem um target que é o app-forum-api:8080. 

Se formos avaliar, você vai ver que esse job app-forum-api está com o scrape_interval de 5s, está herdando a característica da configuração global.

Não estou sobrescrevendo, acho que 5 segundos é o ideal nesse caso para olharmos para uma API, mas você pode customizar isso. 

Isso é a nossa introdução à configuração do Prometheus. 

Vamos ver tópicos mais avançados até o fim do curso, mas, no momento, isso é necessário para que você possa entender o funcionamento dele.

Por último, vamos só dar uma olhada no Docker Compose para você entender que, no momento de subida do contêiner, nós estamos utilizando esses arquivos.

```
 volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
```

O local exato da configuração do prometheus.yml, do arquivo, é em /etc/prometheus/prometheus.yml, e aqui é onde está o TSDB, é um diretório que está saindo da nossa máquina, é aquele diretório que eu tive que dar uma permissão muito aberta para que o usuário do Prometheus pudesse manipulá-lo dentro do contêiner.

Aqui estão as flags. 

Lembra das flags que vimos na interface web? 

Então, aqui tem algumas flags que estão sendo utilizadas que podem ser encontradas na interface web dele.

Então, é isso, foi só uma navegação pelo Prometheus, entendendo algumas configurações dele. Vamos aprofundando, vamos trabalhando mais nisso e olhando para o que interessa no decorrer do curso.

## Subindo o cliente para consumo da API -- automatizado
Nós já temos a nossa API, nós possuímos o nosso database, o nosso cache e o nosso proxy, porém, não temos um client, um consumidor dessa API.

E isso nos leva a um problema porque não temos as métricas sendo alimentadas através desse consumo. 

Nós poderíamos utilizar algum recurso, como a extensão de um browser, para fazer requisições de tempos em tempos, ou seja, atualizações daquela página em que estamos consumindo o endpoint.

Ou poderíamos continuar utilizando o Postman, só que essas duas soluções trariam uma carga de processamento que não é interessante, nesse momento, para a nossa infraestrutura.

Então, a solução é a seguinte: **vamos rodar um contêiner bem simples e esse contêiner vai consumir a nossa aplicação em todos os endpoints que ela possui e, inclusive, gerar alguns erros de tempo em tempo para que tenhamos todas as métricas devidamente alimentadas**.

Para fazer isso, eu quero que você acesse a pasta em que está toda a configuração – no meu caso, está no workdir do Eclipse, eu estou no Linux, e aqui tem dentro tem um diretório chamado client.

### Pasta client

Eu chamo de “diretório”, se você está no Windows, você está mais habituado a ouvir “pasta”, mas já faz esse link, quando eu chamar diretório, é pasta.

Vamos entrar no client, nesse diretório, e aqui dentro eu tenho um Dockerfile e um script Shell, um script em linguagem de Terminal do Linux. Não se preocupa porque o contêiner é Linux, então é esse script mesmo que tem que rodar, sem nenhuma modificação.

Vamos conhecer esse Dockerfile para que você possa entender o que vamos fazer. 

```
FROM debian

USER root

COPY ./client.sh /scripts/client.sh

RUN apt update && \
        apt install curl -y && \
        chmod +x /scripts/client.sh

ENTRYPOINT ["/scripts/client.sh"]

```

Nesse Dockerfile, vamos gerar um contêiner derivado da imagem do debian, da última versão da imagem, por isso que não estou com tagueamento de versão para ele.

O usuário que vai executar vai ser o root. Eu não estou preocupado com segurança nesse momento, então vai ser o root mesmo que vai executar. 

E o que eu vou fazer aqui? 

Vou fazer uma cópia daquele script client.sh para dentro desse path /scripts/client.sh.

Depois disso, eu vou rodar, através do RUN, um contêiner, que é o apt update que vai atualizar a base índice de pacotes do Debian. O Linux funciona por pacotes e sempre vai haver um software que faz a gestão de pacotes.

No caso do Debian, em alto nível, é o apt – Debian e derivados, como o Ubuntu. 

Vamos rodar o apt update que vai sincronizar o que eu tenho de referência de pacotes com o que existe na internet, deixar isso atualizado e depois eu vou instalar um software chamado curl.

O curl vai fazer o papel do browser, é ele que vai fazer a requisição para a nossa API. 

Ele é o client que nós vamos usar. 

E depois disso vou dar um comando chamado chmod (change mode) que altera as permissões desse script, delegando para ele o bit de execução, então ele vai virar um executável.

tem o entrypoint do contêiner que é a chamada que é executada assim que o contêiner sobe, o objetivo de vida desse contêiner vai estar no entrypoint eu é a execução desse script client.sh.

É bem simples essa configuração. 

Está aqui o Dockerfile e vamos dar uma olhada no script, no client.sh. 

```
HOST='proxy-forum-api'

while true
    do
	ENDP=`expr $RANDOM % 3 + 1`
	NUMB=`expr $RANDOM % 100 + 1`
	#TEMP=`expr 1 + $(awk -v seed="$RANDOM" 'BEGIN { srand(seed); printf("%.4f\n", rand()) }')`
        
	if [ $NUMB -le 55 ]; then
	    curl --silent --output /dev/null http://${HOST}/topicos
        elif [ $NUMB -ge 56 ] && [ $NUMB -le 85 ] ; then
	    curl --silent --output /dev/null http://${HOST}/topicos/$ENDP
        elif [ $NUMB -ge 86 ] && [ $NUMB -le 95 ] ; then
	    curl --silent --output /dev/null --data '{"email":"moderador@email.com","senha":"123456"}' \
		 --header "Content-Type:application/json" \
		 --request POST http://${HOST}/auth
        elif [ $NUMB -ge 96 ] && [ $NUMB -le 98 ] ; then
	    curl --silent --output /dev/null --data '{"email":"moderador@email.com","senha":"1234567"}' \
	         --header "Content-Type:application/json" \
	         --request POST http://${HOST}/auth
	else
	    curl --silent --output /dev/null http://${HOST}/topicos/0
        fi

	#sleep $TEMP
	sleep 0.75
done

```

Esse script tem uma variável chamada host, essa variável recebe o nome do contêiner proxy, o proxy-forum-api.

A configuração dessa variável é o nome do contêiner proxy e ele tem um laço infinito, um while. 

Esse while é infinito porque ele está avaliando a condição de true, e true é um dado booleano que sempre vai ser verdade. 

Então, isso nunca vai parar de ser executado, é um laço de repetição infinito.

são declaradas três variáveis. 

A primeira variável (ENDP) vai conter um número aleatório de 1 a 3; a cada execução do while, a cada iteração, essa variável vai ser alimentada com uma configuração de um valor inteiro variando de 1 a 3.

A segunda variável (NUMB) vai fazer a mesma coisa, mas vai ter um valor de 1 a 100 a cada iteração. 

E a terceira variável (TEMP) , também vai receber um valor aleatório, porém, um valor menor que 1, então ela vai receber um float que está entre 0 e 1.

Para que isso? 

Para que tenhamos, primeiro, um tempo a cada iteração que vai ser inferior a 1 segundo; depois disso, para que possamos atingir um dos endpoints que já está configurado, que é o ID de um tópico que sempre vai variar de 1 a 3.

A cada iteração desse laço, eu vou atingir um endpoint, o endpoint /topicos, em um ID específico que vai variar de 1 a 3 a cada execução que eu entrar, na condição específica de fazer uma requisição para ele.

Quanto ao outro número (NUMB), é ele que segue com a lógica de negócio principal desse script simples. 

Basicamente, o que eu faço aqui é uma estrutura condicional de if/else. Esse if vai avaliar se a variável numb, que recebe um valor de 1 a 100, é menor ou igual a 50.

Se ela for menor ou igual a 50, ele vai fazer uma requisição para /topicos, ele vai atingir o proxy-forum-api em /topicos; se for maior ou igual a 51 e menor ou igual que 80, vamos mandar uma requisição para /topicos em um ID que vai estar variando de 1 a 3 a cada iteração.

Caso o numb esteja maior ou igual a 81 e menor ou igual que 90, fazemos uma autenticação com sucesso. Agora, se numb for maior ou igual que 91 e menor ou igual que 95, nós falhamos uma autenticação; se esse valor estiver acima de 95, ou seja, se for de 96 até 100, nós enviamos uma requisição para um ID que não existe e isso gera um erro também.

Então, a condição mais simples de ser atingida é uma requisição para /topicos. 

A segunda condição mais fácil de ser atingida é a que envia uma requisição para um ID que vai variar de 1 a 3 em /topicos.

A nossa terceira condição, que é um pouco mais difícil de ser alcançada, é uma autenticação com sucesso. 

Depois, os erros têm um percentual bem pequeno de chance de ocorrer e eles vão estar alimentando métricas relacionadas à falha de autenticação e as chamadas a endpoints que não existem, que não vão ser encontrados.

Para você entender o que é o random, é um pseudodispositivo, e toda essa lógica que foi feita nesse script bem simples. 

Basicamente, esse Dockerfile vai pegar esse script, vai configurá-lo, com permissões de execução, vai sanar as dependências instalando o curl, e vai chamar esse script no entrypoint de execução.

### Client-forum-api
Saindo desse escopo, temos o nosso Dockerfile. 

```
client-forum-api:
    build:
      context: ./client/
      dockerfile: Dockerfile
    image: client-forum-api
    container_name: client-forum-api
    restart: unless-stopped
    networks:
      - proxy
    depends_on:
      - proxy-forum-api
```

Abrindo esse Dockerfile, você vai notar que o seu está com o último bloco comentado. 

Vamos descomentar esse bloco e eu vou te explicar o que esse bloco está fazendo.

Esse server que vai ser gerado no Docker Compose se chama client-forum-api, ele vai fazer um build. 

O contexto desse build é no diretório client, e lá ele vai encontrar um Dockerfile. Você já sabe o que o Dockerfile faz e o que o script faz também.

Ele vai gerar uma imagem chamada client-forum-api e um contêiner derivado dessa imagem chamado client-forum-api também. Se esse contêiner for derrubado, ele não vai subir.

Todos os contêineres estão com essa configuração porque, em dado momento, vamos realmente derrubar alguma coisa para verificar como que as métricas vão ser alimentadas e, principalmente, para testar os alertas que serão feitos mais para frente.

Ele está dentro da rede proxy e qual é a dependência dele? 

O prometheus-forum-api, o contêiner do Prometheus. 

Por que ele depende do Prometheus? 

Porque temos uma cadeia de dependências que permite somente que um contêiner suba após o sucesso do outro.

Isso começa desde o primeiro service que possuímos aqui, o redis. 

O redis não depende de ninguém, ele tem que subir; após o redis, tem o mysql, que depende do redis; depois, tem o redis-forum-api, que depende do mysql; depois, o proxy, que depende do app; e o prometheus, que depende do proxy; e, por último, o client, que depende do prometheus.

Por hora, essa configuração vai nos auxiliar, porque esse script vai ser executado nesse contêiner e ele, logicamente, vai fazer muitas requisições para a nossa aplicação sem trazer uma carga de CPU ou de consumo de memória notável para a sua infraestrutura, para a sua máquina pessoal, que é o host que está rodando essa stack.

Vamos rodar o comando docker-compose up -d para subir, modo daemon. 

Ele vai fazer o update da base índice de pacotes; vai instalar o software que vamos utilizar, o curl; depois, vai rodar as permissões e vai subir a stack. 

Depois disso, já vamos ter um client consumindo a nossa aplicação e, por fim, vamos ter dados relevantes nas nossas métricas, vamos ter, enfim, insumos para podermos identificar melhor a funcionalidade de cada métrica.

## A anatomia de uma Métrica
Agora que já temos o consumidor da nossa API, vamos conseguir ter uma visibilidade melhor sobre as métricas, mas, para que possamos realmente fazer bom uso do que estamos fazendo, é muito importante que você entenda como é uma métrica para o Prometheus.

No browser, vamos no “localhost/metrics”, vou atualizar, você vai ver que muita métrica foi gerada por conta daquele client que nós rodamos. 

Inclusive, alguns erros 500 vão se detectados, porque o script começou a ser executado pelo client antes de o MySQL estar realmente atendendo às requisições, então gerou uma quantidade de erros 500 em /topicos.

Foi só durante a subida, agora está certo. 

Você vai notar o seguinte, os valores estão aumentando porque a aplicação está sendo consumida.

Se descermos um pouco, vamos ver que /topicos está aqui; 

temos também as métricas de usuários.

Tenho também autenticações com erro. 

Voltando ao que interessa, se olharmos para uma métrica, vou procurar pela métrica de log, que talvez seja a mais simples de explicar no momento.

Vamos pegar essa métrica aqui, **logback_events_total**. 

### metric name
Toda métrica tem três componentes básicos. O primeiro é o metric name, é o nome da métrica. 

Aqui, o metric name que nós temos é o logback_events_total. 

Se eu executo essa consulta no Prometheus, eu tenho alguns retornos.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/7.PNG)

Se eu pegar um desses retornos específicos - o primeiro, por exemplo - e fazer essa consulta, eu tenho só o retorno dessa consulta específica com esses atributos. 

https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/8.PNG

Vamos entender o que são esses atributos e o que é o retorno da consulta.

Então, tenho o metric name, que nesse caso é o logback_events_total, e logo após o metric name tenho labels, rótulos. 

### labels

```
{application="app-forum-api", instance="app-forum-api:8080", job="app-forum-api", level="error"}
```
Esses rótulos vão identificar qual é a série temporal que você quer consultar.

Até agora vimos duas entidades aqui: metric name e labels. 

Podem existir vários labels configurados, eles vão estar nessa configuração similar a uma variável, de chave-valor, {application=”app-forum-api”,instance="app-forum-api:8080",job-"app-forum-api",level="error"}.

### sample
À direita dos labels, eu tenho o sample, que é o resultado dessa consulta, que está aqui, 0. 

Se eu mudar alguns desses labels, por exemplo, eu não quero olhar o level debug do log, eu quero olhar o info: 

logback_events_total{application=”app-forum-api”,instance="app-forum-api:8080",job-"app-forum-api",level="info"}

Vamos fazer a consulta dele, eu tenho 43 como retorno, meu sample como 43.

Já entendemos que existe metric name, que existe label e que existe o sample, que é o resultado da consulta sobre aquela métrica com aquele metric name e aqueles labels específicos.

Isso está fácil de entender. 

Agora nós entramos no ponto de quais são os tipos de dados que uma métrica pode ter – qual tipo de dado é aquela métrica? 

### Tipos de Dados no Prometheus
No Prometheus, temos quatro tipos de dados.

#### Instant Vector
Temos o instant vector, que é um vetor instantâneo; 

#### Range Vector
temos o range vector, que é um vetor de uma série temporal; 

#### Scalar
temos um dado do tipo Scalar, que é um float simples, um ponto flutuante; 

#### String
e temos o string, **que basicamente não é utilizado**, a própria documentação do Prometheus diz isso, que não é comumente utilizado.

Vamos só focar nos três tipos de dados mais utilizados no Prometheus. 

O que é um instant vector? 

É um vetor instantâneo, esse vetor está sendo exibido agora para vocês na tela, ao executar logback_events_total.

Quando eu olho para essa métrica, logback_events_total, ela me retorna um vetor. 

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/9.PNG)

Esse vetor está aqui embaixo, cada uma dessas linhas seria um índice desse vetor. 

Eu teria o índice 0, 1, 2, 3 e 4. Vamos simplificar dessa maneira.

Cada um desses elementos que estão dentro desse vetor **é uma série temporal**. 

Eu tenho aqui 5 séries temporais que estão armazenadas nessa métrica logback_events_total e, quando eu faço uma consulta para essa métrica, esse vetor retorna para mim.

O que gera esses vetores e como eu faço a distinção entre cada um? 

Através de levels, são os labels que vão fazer essa distinção. 

Se olharmos para essa série temporal específica, vou pegar novamente o debug, você vai notar o seguinte.

Qual é a diferença de uma para outra? 

É o level. 

Essa métrica está relacionada ao log, e o log tem níveis de logs específicos que a JVM consegue externalizar através do Actuator e o Prometheus acaba fazendo a interface através do Micrometer.

Nós conseguimos ter acesso à essa informação através de uma métrica com um label de level distinto. 

Então, tivemos, na subida da aplicação, uma quantidade de info, uma quantidade de erros e uma quantidade de warnings. 

Não tivemos log level de debug aqui, nem de trace.

Isso é um instant vector, um vetor de tempo. 

Se você olhar em “Evaluation time”, você vai ver o momento da consulta. 

Então, falamos sobre o primeiro tipo de dado que é o instant vector.

Agora, vamos falar sobre o range vector. 

Para chegarmos nesse ponto do range vector, eu vou tirar tudo e colocar 1 minuto,[1m], entre colchetes - logback_events_total[1m]. 

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/10.PNG)

A partir do momento que eu faço isso, estou trabalhando com o range vector.

Notem que teve uma transformação aqui, eu estou olhando justamente para esse timestamp, posso tirá-lo e refazer a consulta, vai dar na mesma. 

O que estou trazendo é o último minuto dentro desse timestamp.

Esse foi o timestamp de execução da consulta, eu quero o último minuto. 

No último minuto, eu tenho essas informações que voltaram. 

Notem que o range vector não seria uma matriz, não temos um vetor multidimensional, mas temos, em cada série temporal, um “vetor” que traz um valor específico a cada scrape time do Prometheus.

Vamos lembrar a nossa configuração. 

Se eu abrir aqui e for em “Configuration”, você vai ver que o nosso job app-forum-api tem o scrape de 5s. 

Então, a cada 5 segundos o Prometheus vai lá e bate no endpoint para trazer as métricas.

Justamente aqui, em 1 minuto, nós tivemos essa quantidade aqui, 12 consultas, cada uma a 5 segundos, o que equivale a 60 segundos de trabalho do Prometheus buscando métricas.

Alguns estão com valores e outros não, não está dando para pegarmos uma mudança aqui nesses valores, mas normalmente vamos enxergar mudanças.

https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/11.PNG

Aqui tem uma notação meio estranha, que é o unix timestamp. 

Se você quiser entender o que é isso, no Linux é fácil – se você estiver no Windows, deve ter uma forma fácil de você fazer no PowerShell, ou você vai no browser e dá uma "googlada" “unix timestamp” que você vai encontrar um jeito de converter.

Então, quando falamos em range vector, estamos falando de um range de tempo dentro de uma série temporal.

Eu posso facilitar um pouco isso, posso olhar um intervalo específico? 
##### Subquery
Posso, através de uma subquery. 

Eu quero olhar os últimos 5 minutos e quero ver apenas o último minuto. Está aqui, consigo fazer isso: logback_events_total[5m:1m].

Um minuto ficou muita coisa, quero olhar os 30 segundos. 

Também consigo: 

logback_events_total[5m:30s]

Então, dentro dos últimos 5 minutos, eu olhei os 30 segundos.

É bem simples de se trabalhar, vamos ter funções que vamos ver mais a frente que vão aproveitar melhor o range vector. 

Tem um detalhe, estamos vendo uma saída que é tabulada, e se quisermos ir para gráfico?

Não é possível. 

Se você ler isso, você vai ver que **“houve um erro executando a consulta, o tipo ‘range vector’ é inválido para esse tipo de consulta, tente com um Scalar ou um vetor instantâneo”**.

Isso significa o quê? 

Que eu não posso formar um gráfico no Prometheus quando eu tiver uma saída que possui mais de um valor. 

Ou ela tem que ser um instant vector ou um dado Scalar. 

O dado Scalar, só lembrando, é um float.

Vou executar dessa maneira, 

logback_events_total

Eu tive esse retorno, ele formou um gráfico para mim, está bem simples, mas formou o gráfico. 

Aqui eu tenho uma consulta que traz para mim um instant vector. 

Sendo um instant vector ou sendo um dado Scalar, eu consigo retorno.

Voltando, o que faltou é falarmos sobre o Scalar. 

O dado Scalar é, basicamente, um float. 

Posso encontrar qualquer um aqui, por exemplo, deixa eu olhar para alguma coisa de conexões.

hikaricp_connections_idle

Tenho um dado que vai me retornar um valor que é um float simples e que vai poder ser visualizado. 

Se eu for para o modo gráfico, eu consigo enxergar também um gráfico.

Não vamos falar de string porque isso não cabe dentro do nosso conteúdo e dificilmente você vai ver algum caso de uso para string envolvendo o Prometheus. 

Isso pode ser encontrado na própria documentação.

Eu aconselho que você aprofunde mais utilizando essa documentação, vamos estender esse assunto, só que não vamos esgotá-lo. 

Então, acho muito interessante você consumir a documentação e aprofundar um pouco mais.

Agora que entendemos a anatomia de uma métrica, entendemos os três componentes – metric name, label e o retorno da consulta que é o sample.

E entendemos também os tipos de dados que o Prometheus utiliza.

## Tipos de Métricas
Nós abordamos todos esses assuntos e, para complementar isso, é de suma importância que você entenda quais são os tipos de métricas que o Prometheus trabalha.

Eu vou recorrer à documentação oficial. 

Para você chegar na documentação oficial, é bem simples, é só ir em “Help” no painel superior do Prometheus, você abre em outra aba e você vai estar na documentação.

Na documentação, você pesquisa por “metric types”.

https://prometheus.io/docs/concepts/metric_types/#metric-types

O Prometheus trabalha com quatro tipos de métrica.

Tem o tipo “Counter”, “Gauge”, “Histogram” e “Summary”. 

### Counter
Vamos falar primeiro do “Counter". 

**A métrica do tipo counter é uma métrica que é crescente e sempre será incrementada.**

Então, é uma métrica cumulativa. 

Qual é a desvantagem que temos no counter? 

Se a sua aplicação for reinicializada, essa métrica vai zerar, só que ela vai zerar dentro do seu tempo atual de execução da aplicação.

Você vai conseguir consultar os resultados anteriores que estão no TSDB em outra série temporal, sem problema, porém, o valor atual da métrica vai ser zerado.

**Não é aconselhável que você usar o tipo de métrica counter para trabalhar com valores que vão variar, que vão subir ou descer.** 

Sempre use com valores incrementais.

Se formos olhar o tipo counter no endpoint de métricas, você vai ver que tem diversos exemplos. 

```
# TYPE jvm_classes_unloaded_classes_total counter
jvm_classes_unloaded_classes_total{application="app-forum-api",} 1.0
```

Temos aqui a métrica personalizada que nós criamos, auth_user_success_total. 

Posso colocar essa métrica no Prometheus e ela sempre vai ser incrementada.

Eu posso procurar também o erro, nós também criamos uma métrica para a condição de erro, está aí, auth_user_error_total. 

É legal entender por que nós trabalhamos com o counter, já que em um momento vamos ter usuários logados e em outro não.

Nós trabalhamos com essa métrica porque é interessante você ter um histórico e entender, em um período específico, quantos usuários fizeram login, em um range específico de tempo.

No momento atual da nossa aplicação, nós vamos utilizar uma lógica através de alguns operadores e funções que vão trazer para nós o valor exato do momento da consulta.

O counter é justificado para esse uso que nós fizemos por conta de você poder ter uma avaliação de uma janela grande de tempo e entender qual era a média de autenticações que você teve em um período tal e quanto você está tendo hoje.

Esse seria o motivo de termos o tipo de counter para autenticações e erros de autenticação.

Se eu procurar uma métrica, por exemplo, a do log, também é um tipo counter o logback_events_total. 

É legal porque, se você procurar, o próprio Prometheus vai dizer para você qual o tipo da métrica.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/12.PNG)

Então, estamos conversados sobre o que é um counter. 

ele vai contando e fazendo a divisão pelo log level, temos aqui debug, error, info, trace e warn. 

Ao todo, são cinco níveis de log que conseguimos ter e ele vai contando os eventos que se categorizam sobre o nível de log específico em uma série temporal específica.

### Gauge
Já entendemos o que é um tipo de métrica counter, vamos falar sobre o "Gauge". 

Lembra que eu falei que o counter não deve ser utilizado se a sua métrica for variar, se ela vai ter um valor inferior e depois superior, se ela vai subir e descer?

O gauge já trabalha nesse ponto, **ele é direcionado para valores que vão variar no decorrer da execução do seu sistema**. 

Logicamente, o gauge é uma métrica que se encaixa muito bem para que possamos mensurar consumo de CPU, consumo de memória, número concorrente de requisições em um período de tempo específico e fazer comparações.

A utilização do gauge é para valores variáveis. 

Se dermos uma procurada aqui, “gauge”, temos, por exemplo, o estado de threads da JVM. 

Por exemplo, se eu fizer essa consulta: jvm_threads_states_threads{application="app-forum-api",state="runnable"}. Vou tirar o state=runnable porque estamos pegando as métricas que estão em estado runnable – ao todo, são 7.

Se mudarmos aqui, vou procurar por outro state, vou colocar timed-waiting, eu tenho 8. 

Esses valores vão ser modificados conforme a aplicação for consumida, conforme ela tiver chamadas de funções e esse tipo de coisa.

Se procurarmos por outra, eu tenho aqui conexões da JDBC em estado idle. 

Deixa eu encontrar outra, aqui também é sobre thread, buffer. 

Está aqui, contagem de CPU. 

system_cpu_counter, essa métrica eu não acho interessante porque ela pega o número de núcleos de um CPU, então é uma coisa que, no nosso caso, não vamos ter aplicabilidade.

Agora aqui, uma métrica bem legal é a utilização do CPU para o processo que a JVM está executando: process_cpu_usage. 

Esse valor vai subir, vai descer, vai variar conforme a execução da nossa aplicação.

Até aqui, acho que está tranquilo, são dois tipos de métricas bem simples: o counter, o valor incremental – se a aplicação for inicializada, ele é zerado; e o gauge, uma métrica que vai variar, vai sofrer incremento e decremento no decorrer da execução do sistema.

### Histogram
Aí chegamos em uma métrica mais complicada, do tipo "Histogram". 

**O histogram traz observações que estão mais relacionadas à duração e ao tamanho de resposta.**

Ele tem uma configuração de alocação de séries temporais em buckets. 

Esses buckets vão corresponder a algumas regras que vamos definir. 

No nosso caso, nós já definimos, porque criamos uma métrica do tipo histogram quando trabalhamos no application properties e definimos aquela métrica de SLA diretamente no application.prod.properties da aplicação.

```
management.metrics.distribution.sla.http.server.requests=50ms,100ms,200ms,300ms,400ms,500ms, 1s
```

Existem N buckets, cada bucket tem uma configuração que nós setamos lá no application properties, e ele está relacionado à duração de uma requisição e ao tempo de resposta que eu tenho.

Além disso, o histogram traz para nós a soma total dos eventos observados em questão de tempo – quantos segundos, por exemplo – e traz também a contagem de todos os eventos.

Para entendermos um pouco melhor isso, podemos procurar aqui http_server_requests_seconds histogram. 

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/13.PNG)

Abaixo dele, temos os buckets, você vai ver as linhas do bucket, e você vai ver count e sum.

Vou pegar uma específica, pode ser essa mesma, http_server_requests_seconds_bucket. 

No caso, estou distinguindo por label. 

A métrica é a mesma, o que muda são os labels. 

Além de ter os labels distintos para um endpoint específico, eu tenho o bucket definido em uma regra bem lógica. 

Você vai entendê-la agora.

Eu copiei a métrica, que é http_server_requests_seconds_bucket e aqui eu tenho todos esses labels. 

Eu não preciso da maioria deles, eu posso retirar toda essa informação e deixar só status="200",uri="/topicos/(id)", aí tem essa regra que é uma condição, le.

**“Less or equal”, “menor ou igual”.** 

Se olharmos para esse valor aqui, 0.05, temos que entender que esse valor está em milissegundos, então tenho 50 milissegundos. 

Se eu executar (
  http_server_requests_seconds_bucket{status="200",uri="/topicos/(id)",le="0.05"}
), ele trouxe “409”, então tenho 409 requisições nesse momento, para “topicos/id”, que estão menores ou iguais a 50 milissegundos.

Colocamos le="0.1", a 100 milissegundos tem mais, são 411; 

a 200 milissegundos, 412; 

a 300, também “412”; 

a 500, do mesmo jeito; 

e com 1 segundo, le="1.0", temos até 1 segundo, 415 requisições.

Se eu tirar esse le=”1.0” e executar, ele vai trazer todas as séries temporais que eu tenho relacionadas aos buckets. 

Aqui é acima de 1 segundo, é infinito e além, esse +inf, é infinito, é acima de 1 segundo.

Então, uma métrica do tipo histogram vai nos auxiliar a fazer esse tipo de medição que é importantíssima para entendermos o tempo de resposta da nossa API e, principalmente, se estamos cumprindo o nosso SLA.

Na verdade, se estamos cumprindo com o nosso SLO, que é o nosso objetivo de nível de serviço que vai ser mensurado com base no nosso SLA. 

Aqui, entraria o nosso indicador do nosso nível de serviço que, de fato, é a métrica.

Voltando, eu posso fazer uma modificação, posso colocar a soma, sum, qual a soma de todos os segundos que eu tenho dessas requisições: 

http_server_requests_seconds_sum{status="200",uri="/topicos/(id)"}.

Está aqui, trouxe um valor que é difícil de entender sem usar uma função. 

E posso trazer a contagem, o número total, sem distinções relacionadas a buckets (
  http_server_requests_seconds_count{status="200",uri="/topicos/(id)"}
). 

Tenho 432 requisições.

#### histogram quantile
O histogram também é uma métrica cumulativa. 

Além de ela ser cumulativa, ela tem uma função que é bem interessante utilizarmos, que é o histogram_quantile, que trabalha com quantiles, então você consegue fazer especificações de quantiles através dessa função e trabalhar bem com ela. Vamos fazer isso mais a frente.

### Summary
O summary é uma métrica muito similar ao histogram, a própria documentação diz isso, só que ele é mais usualmente utilizado para você ter a duração de uma requisição e o tamanho da resposta que você tem para uma requisição.

Além disso, você consegue ter tanto o somatório quanto a contagem – o somatório de segundos da métrica e a contagem total de eventos que foram captados na métrica.

#### Quantiles configuraveis
Uma coisa bem legal é que você também pode calcular quantiles configuráveis de uma forma customizada. 

Você consegue fazer isso dentro de uma janela de tempo.

## Seletores, Modificadores e Funções

No topico anterior, nós falamos sobre tipos de métricas. 

Agora, já vamos entrar em outro assunto que está relacionado a seletores, agregadores e funções. 

Vamos ter uma prévia sobre isso para você entender como você pode manipular uma métrica e obter o resultado que você precisa.

Para começarmos a falar desse assunto, vou trabalhar com essa métrica que é o **http_server_requests_seconds_count**. 

Eu tenho essa métrica, se eu executar a consulta no Prometheus, ele vai me trazer esse instant vector que tem várias séries temporais, mas vou fazer uma filtragem.

Vou colocar o application="app-forum-api", vou colocar qual é o método que vamos olhar, method="GET". 

Vamos também fazer uma verificação no status="200", requisições que retornaram sucesso. 

vamos executar, http_server_requests_seconds_count{application="app-forum-api",method="GET",status="200"}.

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/14.PNG)

Executando essa consulta, o nosso retorno trouxe para nós um endpoint chamado actuator/prometheus; 

trouxe também o /topicos e o topicos/{id}. 

O actuator/prometheus não é um endpoint consumido pelos nossos usuários, então ele não nos interessa nessa visualização.

O que vamos fazer? 

#### seletor de negação (!=)
Vamos trabalhar com um seletor de negação. 

Vou colocar na uri tudo que for diferente de actuator/prometheus, http_server_requests_seconds_count{application="app-forum-api",method="GET",status="200",uri!="/actuator/prometheus}. 

Melhorou um pouco a situação porque tive agora o /topicos e o /topicos{id} retornando para mim a contagem de requisições.

Só que existe também uma autenticação que ocorre nessa aplicação e ela é através do método “POST”, então eu preciso mudar o meu seletor para que consiga obter esse resultado.

#### seletor OU LOGICO (=~)
Para fazer isso, é bem simples. Quando vou trabalhar com o "ou" lógico aqui dentro do Prometheus, da linguagem PromQL, eu tenho que mudar o meu seletor e usar esse seletor com acento que, basicamente, é um seletor que suporta uma espécie de regex.

Vou colocar method=~"GET|POST" e vou executar. Legal, tenho aqui esse retorno que é o método POST com sucesso, status="200", em /auth.

**Isso são seletores. **

Vamos imaginar a seguinte situação, vamos imaginar que você tem uma aplicação, você tem uma API e ela retorna outros códigos além do 200, ela retorna códigos da família do 200 também. 

Como vamos resolver isso?

Vamos trabalhar novamente com o nosso seletor com acento. Ao invés de usarmos 200, vamos usar um coringa, que é o ponto. 

**O ponto é um coringa que equivale a qualquer caractere – como no Unix ou Linux, você utiliza o asterisco.**

Está aqui, status=~"2..". 

Você vê que não mudou nada a consulta está certa. 

Vamos expandir isso, vamos imaginar que, de repente, a sua API está por detrás de um CDN, tem um redirecionamento, um proxy, então tem também a família 300. Vamos colocar status=~"2..|3..".

Até agora, o que fizemos? 

Estamos trabalhando com os métodos GET e POST, estamos olhando para status 200 ou 300, estamos olhando para qualquer URI que não seja /actuator/prometheus.

Agora vamos imaginar no caso em que você quer simplesmente pegar os erros. 

Eu preciso saber o que não está dando certo, o que podemos mudar? 

Vamos no seletor e vamos criar um seletor de negação que suporta o nosso regex: status!~"2..|3.."

Vou trabalhar com esse seletor e aqui tenho o retorno dos erros que captei na minha aplicação. 

Teve erro 500, 404, 400 e mais outro 500 aqui embaixo. Dessa forma, conseguimos trabalhar dentro de uma métrica fazendo uma consulta customizada para pegarmos o resultado que é realmente importante para nós.

#### Aglutinaçao de resultados
Saindo dessa questão de seletores, um detalhe, eu posso também fazer uma aglutinação de resultados. 

Se eu olhar para o “Evaluation time”, esse é o timestamp da consulta que eu fiz, estou pegando essa informação com uma restrição de tempo.

Vamos imaginar que eu quero olhar o que aconteceu, em termos de número, no último 1 minuto. 

Vou trabalhar com o modificador offset e vou setar 1m, http_server_requests_seconds_count{application="app-forum-api",method=~"GET|POST",status!~"2..|3..",uri!="/actuator/prometheus} offset 1m

Vou executar, ele pegou o resultado do último minuto. 

Estamos fazendo uma observação, em uma série temporal, de 1 minuto. 


Mas, basicamente, esse conteúdo está em help e em “Querying > Basics”. 

Se procurarmos aqui, o modificador offset, está tudo na documentação e eu vou deixar o link para vocês.

### funçoes

Uma coisa que é bem legal é começarmos a trabalhar com funções. 

Nós já entendemos o que são seletores, já trabalhamos com um modificador para fazer uma aglutinação, porém, podemos ir além disso e trabalhar com funções que, basicamente, é o recurso que mais vamos utilizar daqui para frente na hora de compor as nossas métricas.

Vamos imaginar o seguinte, eu preciso saber a taxa de crescimento de uma métrica específica. 

Vamos entender que essa métrica está relacionada ao número de requisições. 

Então, o que eu vou fazer?

Não me importa qual é o método, qual é o verbo HTTP, e não me importa qual é o status. 

Tenho http_server_requests_seconds_count{application=”app-forum-api", uri!=”/actuator/prometheus”}.

Tenho esse resultado, porém, vamos imaginar que eu quero olhar 1 minuto, vamos olhar para [1m]. 

O que eu vou ter é um range vector que vai trazer para mim um resultado correspondente a cada scrape que o Prometheus executou no endpoint.

Eu não tenho, de forma visível, qual foi a taxa de crescimento dessas séries temporais em 1 minuto, eu só tenho os valores divididos. 

#### funçao increase()
Para facilitar a nossa vida, podemos utilizar a função increase.

**Com a função increase, eu pego a taxa de crescimento** que eu tive no meu último minuto. 

É interessante entender que eu ainda continuo tendo um instant vector de retorno, increase( http_server_requests_seconds_count{application=”app-forum-api", uri!=”/actuator/prometheus”}[1m]).

Basicamente, no último minuto, o que eu tive de taxa de crescimento em /tópicos foi 41 requisições; em tópicos/(id), 22 – estou fazendo um arredondamento meio brusco, mas dá para entender; 7 requisições em auth; não tive nenhum erro 500 em /topicos, tive 404 em tópicos/(id) e nenhum erro 500 em /topicos.

Agora, vamos imaginar o seguinte: eu estou precisando saber qual o foi o número de requisições que eu tive, e o increase está trazendo para mim essa taxa de crescimento, mas não está somando esse valor.

Eu preciso ter uma agregação de conteúdo para que eu tenha o total. 

#### funçao sum()
Vamos trabalhar com o sum, que é um agregador. 

Vou utilizar o sum para pegar o resultado do increase e trazer o número específico de requisições do último minuto, que foram 76 requisições.

Inclusive, eu consigo ir para a parte gráfica e ter a visibilidade de uma métrica já "plotada" para mim. 

É bem interessante entender como podemos fazer agregações e ter o resultado de uma métrica, bem conciso.

Além disso, vamos imaginar que estamos na seguinte situação: vamos mudar isso para [5m] e agora, diferente de obter a taxa de crescimento, vamos imaginar que eu preciso saber quantas requisições por segundo eu recebi nos últimos 5 minutos.

Se eu executar a consulta, vai vir um range vector gigantesco com esse espaço de 5 minutos, uma vez que cada scrape que nós temos é de 5 em 5 segundos.

Vem muita informação, mas eu preciso saber por segundo. 

Como vou trabalhar com isso? 

#### funçao irate()
Vou utilizar a função irate. 

Com a função irate, eu consigo pegar o número de requisições por segundo que eu tive em cada um desses endpoints e me vem esse resultado aqui.

**O irate basicamente olha os dois últimos data points coletados em relação ao momento da sua consulta. **

Então, não é uma função que eu vou utilizar para criar uma métrica para a SLI, por exemplo.

Não, porque é uma função para eu saber, em um momento exato, em um trecho curto de observabilidade, um dado que eu preciso. No nosso caso, o número de requisições.

Se eu levar isso para a parte gráfica, está aqui, ele coloca cada um dos endpoints que ele conseguiu coletar e temos esse retorno já "plotado" em forma de gráfico. 

Eu poderia fazer uma agregação utilizando sum? 

Poderia, eu posso agregar esse resultado.

Basicamente, se eu olhar para esse resultado, ele vai dizer que, por segundo, eu tive uma média de 1.2 requisições a cada segundo; no gráfico, quando fazemos a agregação, o resultado é esse.


# Monitoramento: Prometheus, Grafana e Alertmanager

## Subindo o Grafana

Vamos fazer algumas configurações após a subida do contêiner e essa você realmente vai precisar executar. 

O primeiro ponto, estou no meu workdir, estou no path em que está o meu Docker Compose. 

Vou criar um diretório chamado grafana. 

Esse diretório, essa pasta vai servir como um volume para o contêiner, e vai ser onde o Grafana armazena suas informações.

Após criar esse diretório, vou aplicar um chmod para mudar as permissões dele para 777, então chmod 777 grafana.

Quando fazemos isso, damos esse chmod, nós estamos atribuindo permissões muito abertas, muito inseguras para esse diretório. Porém, estamos falando somente do diretório grafana.

Se eu olhar dentro do diretório grafana, ele está vazio, não tem nada. 

Se eu olhar para o diretório em si, vocês vão ver que qualquer um pode executar, escrever e ler o diretório.

Por que eu tenho que fazer isso? Quando o contêiner do Grafana subir, ele vai ter um usuário próprio que vai ter que usar esse diretório porque ele vai ser montado como volume dentro do contêiner e, se ele não tiver permissões para entrar nesse diretório e escrever, o contêiner não vai subir.

Tendo feito isso, vamos abrir o nosso docker.compose.yml. 

Você tem o contêiner do prometheus e você tem o contêiner do client que depende do proxy – o prometheus depende do proxy e o client depende do prometheus. Temos essa dependência intercalada de contêineres.

Vamos respeitar isso para o Grafana. 

```
grafana-forum-api:
    image: grafana/grafana
    container_name: grafana-forum-api
    volumes:
      - ./grafana:/var/lib/grafana
    restart: unless-stopped
    ports:
      - 3000:3000
    networks:
      - monit
    depends_on:
      - prometheus-forum-api
```

Vou pegar esse conteúdo e vou colar entre o conteúdo do prometheus e do client. 

O que ele diz? 

Que vamos ter um service, que no caso um contêiner chamado grafana-forum-api; ele vai trabalhar com a imagem corrente mais atual do Grafana; o nome do contêiner vai ser grafana-forum-api; ele vai ter como volume esse diretório grafana que eu criei e apliquei aquela permissão; e ele vai ser montado dentro do contêiner /var/lib/grafana.

O esquema de restart é igual ao dos outros contêineres. 

Se derrubarmos esse contêiner, ele não sobe sozinho, isso é proposital. Se tivermos uma falha, não vamos entrar em loop e ficar tentando subir o serviço, e também, em casos em que derrubemos um contêiner para forçar uma situação de erro, ele não vai subir no automático.

Aqui está a porta, o Grafana roda na porta 3000 TCP e vai fazer bind de porta para o seu docker host, para a sua máquina. Então, em “localhost:3000”, você vai acessar o Grafana.

Ele está na network: monit, que é a mesma do Prometheus, e ele depende do prometheus-forum-api. Essa é a configuração do Docker Compose para subir essa stack.

A alteração a mais que eu vou fazer é na dependência do client. 

O client vai subir após o grafana agora. 

É bom porque dá um tempo para o client enviar as requisições para a API.

O client fala com a API através do proxy, ele envia requisições, só que ele sobe antes que o MySQL esteja devidamente configurado. 

No esquema de dependências, se olharmos aqui, você vai ver que temos o redis subindo primeiro; o mysql sobe depois do redis e depende do redis para subir; o app depende do mysql, por sua vez, para subir – aqui está a dependência dele, mysql-forum-api.

O proxy depende do app para subir, da API; e o prometheus depende do proxy. Agora, o grafana depende do prometheus e o client depende do grafana.

Sem essa configuração, do jeito que estava antes, apesar do contêiner do mysql subir, ele ainda não estava com a base configurada e o client já enviava requisições, o que resultava em alguns erros 500 que vocês podem observar nas métricas anteriores que nós verificamos.

Inclusive, você vai conseguir verificar nas suas métricas que isso vai acontecer. Aqui, nós coibimos um pouco dessa falha porque o contêiner do client só vai começar a disparar suas requisições depois que o grafana tiver subido e isso dá um tempo a mais para o MySQL se organizar e estar bem das pernas nessa hora.

Vou salvar o conteúdo e vou rodar o comando docker-compose up -d para ele subir em modo daemon.

Ele vai subir toda a stack naquela ordem de dependências que está no Docker Compose e, por último, ele vai subir o Grafana. 

Para validar isso, vou abrir o browser. 

Em primeiro ponto, vou no “localhost/metrics” para que possamos verificar. Aqui, o erro 500 que eu falei que conseguíamos observá-lo. Por que continuamos observando o erro 500? Estamos usando um volume no Prometheus e o TSDB está sendo armazenado, então temos essa referência histórica.

Se eu procurar por requisições 200, eu tenho o /actuator/prometheus e agora o meu client já começa a atingir o /auth, /topicos e /topicos/{id} está em algum lugar também – não estou enxergando, deixa eu atualizar mais uma vez porque ele também vai bater em /topicos/{id} daqui a pouco.

Já estamos consumindo a aplicação, isso já está sendo refletido nas métricas. Se eu acessar o “localhost:9090”, eu vou cair na interface do Prometheus lá no "Graph”. Aqui, é o Expression Browser, é o navegador de expressão. Vamos agora para o Grafana. O Grafana vai estar em “localhost:3000”, como falei anteriormente.

No primeiro acesso ao Grafana, o login é “admin” e senha “admin”, ele vem default com essa configuração. Autentica e, assim que você autenticar, ele vai pedir para você criar um novo password, uma nova senha. Vou colocar “Alura”.

Vou salvar e pronto, autenticamos no Grafana. Essa é a interface do Grafana. 

### configurar um datasource
A primeira coisa que vamos fazer agora que o Grafana subiu é configurar um data source.

Isso é rápido, vamos no painel à esquerda, no símbolo da engrenagem, em “Configuration > Data sources”. 

O que é um data source? 

É uma origem de dados do Grafana. 

Ele pode ter diversos data sources, pode ser o Splunk, pode ser o CloudWatch, pode ser o Prometheus etc.

Vamos adicionar no “Data sources”, o primeiro da lista já é o Prometheus. Vou selecioná-lo, o “Name” está “Prometheus”, vou colocar que ele está em “http://prometheus-forum-api:9090” TCP.

Esse é o endereço porque ele vai falar com o contêiner. Se você colocar “localhost”, o Grafana vai achar que ele é o próprio endpoint de métricas do Prometheus, e não é.

Aqui você não mexe em mais nenhuma opção, vem e roda um “Save & test”.

Se eu procurar em “Data sources”, vou ver que já tenho o Prometheus configurado. 

### folders
A outra configuração rápida que vamos fazer é, no painel à esquerda, no símbolo do “+”. Vamos entrar em “Folder”.

Vamos criar uma pasta e toda a nossa configuração de dashboard vai ficar nessa pasta. O nome da pasta vai ser “forum-api”, vou criar a pasta e posso criar o meu dashboard dentro dessa pasta.

### dashboards
Vou em “Create dashboard > Dashboard settings” e no nome do dashboard vou colocar “dash-forum-api”. Não vou colocar descrição, nada disso. Vou dar um “save”, vou salvar com as tags e pronto, está feito.

Essa é a subida do Grafana e a configuração mais básica, que é a seleção de um data source e a criação de uma pasta em que o nosso dash será configurado.

## Variáveis e métricas Uptime e Start Time
### variaveis

dashboard settings >> variables >> add variable

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/15.PNG)

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/16.PNG)

### dashboard UPTIME
```
process_uptime_seconds{application="$application", instance="$instance", job="app-forum-api"}
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/17.PNG)

### dashboard START TIME
```
process_start_time_seconds{application="$application", instance="$instance", job="app-forum-api"} * 1000
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/18.PNG)

## Métricas Logback e JDBC Pool

### dashboard Logback (LOGS)
```
increase(logback_events_total{application="$application", instance="$instance", job="app-forum-api", level="warn"}[5m])
```
```
increase(logback_events_total{application="$application", instance="$instance", job="app-forum-api", level="error"}[5m])
```
![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/19.PNG)

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/20.PNG)

### dashboard JDBC pool (connections)
```
hikaricp_connections{application="$application", instance="$instance", job="app-forum-api"}
```
![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/21.PNG)

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/22.PNG)

## Métricas Logged Users e Auth Errors

### dashboard Logged users
```
increase(auth_user_success_total [1m])
```
![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/23.PNG)

### Auth Errors
```
increase(auth_user_error_total[1m])
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/24.PNG)


## Métricas Connection State e Connection Timeout

### Connection State
```
hikaricp_connections_active{application="app-forum-api", instance="app-forum-api:8080", job="app-forum-api", pool="HikariPool-1"}
```

```
hikaricp_connections_idle{application="app-forum-api", instance="app-forum-api:8080", job="app-forum-api", pool="HikariPool-1"}
```

```
hikaricp_connections_pending{application="app-forum-api", instance="app-forum-api:8080", job="app-forum-api", pool="HikariPool-1"}
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/25.PNG)

### Connection Timeout
```
increase(hikaricp_connections_timeout_total{application="app-forum-api", instance="app-forum-api:8080", job="app-forum-api", pool="HikariPool-1"}[1m])
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/26.PNG)

## Métricas Error 500 e Error Rate

### error 500
```
sum(increase(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus", status="500"}[1m]))
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/27.PNG)

### Error rate
```
sum(rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus", status="500"}[5m])) / sum(rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus"}[5m]))
```

```
sum(rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus", status="400"}[5m])) / sum(rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus"}[5m]))
```

```
sum(rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus", status="404"}[5m])) / sum(rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus"}[5m]))
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/28.PNG)

## Métricas Total Requests e Response Time

### Total Requests
```
sum(increase(http_server_requests_seconds_count{application="app-forum-api", instance="app-forum-api:8080", job="app-forum-api", uri!="/actuator/prometheus"}[1m]))
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/29.PNG)

### Response time
```
rate(http_server_requests_seconds_sum{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus"}[1m]) / rate(http_server_requests_seconds_count{application="app-forum-api",instance="app-forum-api:8080",job="app-forum-api", uri!="/actuator/prometheus"}[1m])
```

![](https://github.com/luizClaudioMendes/Observabilidade-coletando-m-tricas-de-uma-aplica-o-com-Prometheus/blob/main/imagens/30.PNG)