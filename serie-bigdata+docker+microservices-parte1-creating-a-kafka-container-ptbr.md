# Parte 1 - Criando um Container Docker para o Kafka

## Sobre a série
Essa é uma série de 3 artigos sobre **Docker**, **Microservices** e **BigData**, publicarei um artigo por semana, todas as segundas. 
Nessa pequena série irei demonstrar o uso do docker com bigdata construindo um feeder de cotações de cryptomoedas.

## Sobre esse artigo
Nesse primeiro artigo iremos criar um container docker com o Kafka, para quem não conhece o Kafka ele é uma espécie de gerenciador de filas (para ficar mais simples a analogia)
que trabalha com streams de dados, sendo uma peça fundamental para bigdata trabalhando na ingestão de dados da camada fast data layer, e possui basicamente as seguintes capacidades:

## Conhecendo um pouco do Kafka

Capacidades do Kafka:
- Publisher e Subscriber para stream de dados, semelhante a um gerenciador de filas;
- Armazena stream de dados de maneira durável com tolerãncia a falhas;
- Processa stream de dados conforme eles ocorrem

Por ser escalavel, resiliente, tolerante a falhas ele é hoje utilizado nas maiores empresas que trabalham com bigdata, como exemplo o Spotify, além de fazer parte das mais utilizadas distribuições para bigdata como Cloudera e Hortonworks.

Para atender essas capacidades ele conta com os seguintes recursos:
- Trabalha em cluster de um ou mais servidores que podem ser expandidos para multiplos datacenters;
- O cluster Kafka armazena os stream de dados em categorias conhecidas como tópicos;
- Cada mensagem consiste intermanete de uma chave, um valor e um timestamp

Ele está dividido basicamente nos seguintes blocos ou APIs:
- Producer: que permite a publicação de mensagem nos tópicos;
- Consumer: que permite consumir as mensages dos tópicos;
- Stream Processor: que permite o processamento de stream de dados de um ou mais streams de entrada, processando-os e produzindo um stream de dados de saida em um ou mais tópicos;
- Conectores: permite criar e executar producers ou consumers reutilizáveis que conectam tópicos Kfaka a uma ou mais aplicações;

### API´s do Kafka
![API´s do Kafka](https://github.com/fnldesign/tech-articles/00-kafka-apis.png)

   
Para quem deseja conhecer um pouco melhor o Kafka coloco abaixo duas referências sobre o mesmo:
- [Site do Kafka](https://kafka.apache.org/)
- [Introdução ao Kafka](https://kafka.apache.org/intro)
- [Tutorial sobre Kafka do TutorialPoint](https://www.tutorialspoint.com/apache_kafka/index.htm)

## Docker
O docker é uma plataforma que permite a criação, teste e implantação de aplicações através de imagens padronizadas conhecidas como containers.

Cada container roda em cima de uma maquina fisica ou virtual, o container possui tudo o que a aplicação precisa incluindo bibliotecas, ferramentas do sistema, código e runtime ou código executavel da aplicação.

O objetivo aqui no artigo não é aprofundar no docker, mas utilizaremos alguns comandos basicos para criar e executar nossa aplicação através de containers.

Para conhecer melhor sobre o docker:
- [Site oficial do Docker](https://www.docker.com/)
- [Visão Geral do Docker](https://www.docker.com/what-docker)
- [Documentação Oficial do docker](https://docs.docker.com/)
- [Docker na Amazon](https://aws.amazon.com/docker/)
- [Referência de Comandos Básicos](https://docs.docker.com/engine/reference/commandline/docker/)

## HandOn - Mão na massa
Dada essa pequena introdução vamos a parte prática. Vamos baixar, iniciar e executar um container docker com uma imagem do Kafka, distribuição do Spotfy.
Logo depois vamos criar um tópico no Kafka e iniciar um Producer para postarmos mensagens e um Consumer para consumirmos as mensagens enviadas.

O Kafka disponibiliza vários métodos e apis para o utilizarmos, bem como ferramentas de linha de comando que iremos utilizar aqui. Veja mais em [Kfaka Clients](https://cwiki.apache.org/confluence/display/KAFKA/Clients)

### Baixando e Iniciando o Container
Os comandos abaixo irão baixar e executar um container com o Kafka e Zookeeper, mapeados para portas padrões 2181 (Zookeeper) e 9092 (Kafka).
```
docker pull spotify/kafka
docker run -d -p 2181:2181 -p 9092:9092 --env ADVERTISED_HOST=kafka --env ADVERTISED_PORT=9092 --name kafka spotify/kafka
```
#### Vamos a explicação dos comandos

O primiero comando (```docker pull spotify/kafka```) baixa a imagem do container Kafka do Spotfy, enquanto o segundo comando (```docker run```) executa o container em background ```opção -d```.

Já as opções ```-p``` definem um mapeamento entre as portas internas do container e as externas do sistema operacional host, ```-env``` definem os valors para váriáveis de ambiente e ```-name``` define o nome do host do container, depois precisamos adicionar esse host name no arquivo host local para que possa ser enxergado pelo host e pelos demais processos e containers. 

Vamos colocar tudo em pé e depois veja como fazer isso na sessão [Ajustando as variáveis de ambiente](./ddd).

`ADVERTISTED_HOST` foi configurado como `kafka`, essa configuração permitirá que outros containers executem os Producers and Consumers de maneira separada e independente.

Configurando `ADVERTISED_HOST` para `localhost`, `127.0.0.1`, ou `0.0.0.0` irá trabalhar bem somente se os Producers e Consumers forem iniciados dentro do container `kafka`, ou se você estiver utilizando o DockerForMac ou DockerForWindows (como eu) e voce quiser rodar os Producers e Consumers do OSX ou Windows respectivamente. 

Mesmo assim você deve mapear o hostname do container a um ip externo ha maquina host, conforme explicado na sessão [Ajustando as variáveis de ambiente](./ddd). Caso contrário ao executar os Producres e Consumers mais a frente você receberá uma erro de `kafka.common.LeaderNotAvailableException`. 

No entanto, esses casos de uso de executar com `localhost` são bem menos interessantes, por isso vamos iniciar Produtores e Consumidores de outros containers.

Em um ambiente produtivo não devemos utilizar localhost, mas sim criar uma rede com o docker isso também será explicado mais a frente na sessão [Configurando uma rede para o docker](./ddd).

Precisamos usar um endereço IP ou nome de host para que o serviço `kafka` seja acessado de outro container. O endereço IP não é conhecido antes do início do contêiner, portanto, temos que escolher um nome de host e eu escolhi o `kafka` neste exemplo.

[Por que estou utilizando a distribuição Spotify?](https://github.com/spotify/docker-kafka#why)

### Criando um tópico
```
docker exec kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

O comando ```exec``` executa um comando no container nesse caso estamos chamado o shell script ```kafka-topics.sh``` que manipula os tópicos do Kafka, os demais parâmetros definem qual o endereço do Zookeeper, o número de particões e o nome do tópico.

saida:
```
Created topic "test".
```

### Listando os tópicos
```
docker exec kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

saida:
```
test
```

### Iniciando um Producer (em uma nova janela)
O comando abaixo utilizará o shell script `kafka-console-producer.sh` para iniciar um Producer para enviarmos mensagens ao Kafka que criamos anteriormente.

Esse comando irá executar um instância do `spotfy\kfaka` linkado ao serviço `kafka` criando anteriormente, iniciará um Producer, e aguardará a digitação das mensagens ou Streams separados por quebra de linha até que você aborte o processo, que destruirá o container do Producer.
Ou seja iniciando essa instancia do container você poderá enviar mensagens para o tópico Kfaka apenas digitando as mensagens e enviando-as digitando enter.

```
docker run -it --rm --link kafka spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic test
```

#### Vamos a explicação do comando

O comando `docker run` já conhecemos e sabemos que ele é utilizado para executar um comando dentro de um container do docker, então vamos as novidades.
O parâmetro `-i`faz o docker rodar de modo iterativo ou seja permite entrada de comandos diferentemente da opção `-q` apresentada anteriormente, já a opção `-t` em conjunto com a `-i` emula um console de terminal TTY, ou seja as duas opções em conjunto nos permitem a entrada de comandos para o Producer como em um terminal padrão.

A opção `--rm` faz com que o container seja removido automaticamente quando fechado.

Já a opção `--link kafka spotify/kafka` faz com que essa instância nova esteja relacionado ao servidor do Kfaka iniciado nos passos anteriores.

Em termos práticos esse conjunto de comandos inicia um contain com um Producer do Kafka que nos permite digitar mensagens para o tópico `test`especificado e envie as mensagens quando damos `ENTER´.
Para finalizar o container pressionamos `CTRL+C` o que para o processo e finaliza o container liberando os recursos do mesmo.

Antes de experimentar os comandos insira o host `kafka` associado aos IPs `127.0.0.1` e `0.0.0.0` para não ocorrer a exceção `kafka.common.LeaderNotAvailableException`.

**Experimente agora**


### Iniciando um Consumer (em uma nova janela)

Para consumirmos as mensagens enviadas pelo Producer criado no passo anteriore precisamos de um Consumer, através do comando abaixo, de maneira similar ao Producer vamos iniciar um Consumer em uma nova janela de console para verificarmos a chegada das mensagens.

Esse comando irá iniciar uma nova instância do container `spotify/kafka` ligado ao serviço `kafka` criado, iniciar um Consumer e mostrar as mensagens enviadas através do Producer para o tópico `test`, de forma similar ao Producer com a opção `--rm` o Consumer será finalizado quando pressionarmos `CTRL+C`.

As opções do Kfaka `--from-beginning` fazem com que o Consumer consuma todas as mensagens do tópico desde o inicio, depois veremos como consumir as mensagens de onde paramos com `offset`. 

```
docker run -it --rm --link kafka spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test --from-beginning
```

**Experimente agora**

#### Enviado e Recebendo Mensagens
Envie algumas mensagens do Producer e digite `ENTER`. As mensagens devem aparecer na janela do Consumer.


### Próximos passos

Como próximos passos no próximo artigo implementaremos um microservice que obterá as cotações de crpyptomoedas e enviará para os tópicos kafka que iremos criar.

Enquanto isso estude as opções de linha de comando do docker, bem como estude mais a fundo o Kafka, nessa parte não exploramos os recursos de particionamento e agrupamento de mensagens, nem a replicação de mensagens dada pelo recurso do `replication factor`.

Dando uma amostra aqui desses recursos indispensáveis e muito importantes do Kafka coloco a imagem abaixo para termos uma ídéia de como o Kafka é poderoso, resiliente e escalável.

#### Anatomia de um Tópico
![Anatomia de um Tópico](https://github.com/fnldesign/tech-articles/01-anatomy-of-a-topic.png)

#### Group ID
![Group Id](https://github.com/fnldesign/tech-articles/02-consumer-groups.png)

Esses recursos podem ser estudados mais a fundo na [Introdução do Kafka] (https://kafka.apache.org/intro).

Até o próximo artigo!!

Aguardo sugestões, opniões, dúvidas e demais contribuições para expandir e discutir esses temas. That´s All Folks!! :sunglasses:
