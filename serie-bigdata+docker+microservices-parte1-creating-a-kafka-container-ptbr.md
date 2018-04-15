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
- [Referneica de Comandos Básicos](https://docs.docker.com/engine/reference/commandline/docker/)

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

O primiero comando baixa a imagem do container Kafka do Soptfy, enquanto o segundo executa o container em background ```opção -d```. Já as opções ```-p``` definem um mapeamento entre as portas internas do container e as externas do sistema operacional host, ```-env``` definem os valors para vária´ves de ambiente e ```-name``` define o nome do host do container, depois precisamos adicionar esse host name no arquivo host local para que possa ser enxergado pelo host e pelos demais processos e containers.

[Por que estamos utilizando a distribuição Spotify?](https://github.com/spotify/docker-kafka#why)

### Criando um topico
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




  
