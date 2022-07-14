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
  - [](#)


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

Dentro do prometheus-grafana, você vai no subdiretório app, em que existe um arquivo com o XML que o Eclipse vai reconhecer automaticamente como sendo o arquivo de um projeto.

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

## 
