# Documentação do Projeto – Docker + Redis

## 1. Integrantes

  * Felipe Werneck
  * João Francisco

## 2. Explicação

Redis (REmote DIctionary Server) é um armazenamento de chave/valor NoSQL de código aberto, em memória, utilizado principalmente como cache de aplicação ou banco de dados de resposta rápida. É comumente usado para:

Análise em tempo real: pode processar dados com latência inferior a um milissegundo, sendo ideal para análises em tempo real, campanhas de publicidade on-line e processos orientados por inteligência artificial.

Aplicações baseadas em localização: simplifica o desenvolvimento de aplicações e serviços baseados em localização, fornecendo indexação, conjuntos e operações geoespaciais.

Armazenamento em cache para bancos de dados: lida com grandes quantidades de dados em tempo real, fazendo uso de seus recursos de armazenamento de dados in-memory para ajudar na compatibilidade com construções de banco de dados altamente responsivas. O armazenamento em cache permite menos acessos ao banco de dados, o que ajuda a reduzir o volume de tráfego e as instâncias necessárias.

O Docker isola os ambientes para aplicativos e serviços que são executados dentro de contêineres. O isolamento significa que é possível empacotar, criar e enviar imagens do Redis que funcionam independentemente do sistema operacional do host, o que facilita o desenvolvimento e a execução de aplicativos Redis dentro do Docker. Além da facilidade de uso, essa abordagem também oferece segurança, flexoflexibilidade e confiabilidade. Para execução do projeto, foi compreendido os seguintes comandos:

* `docker compose up -d`: sobe o serviço Redis em segundo plano (modo "detached").
* `docker ps`: lista todos os containers em execução no momento.
* `docker logs <container>`: exibe os logs do container (para depuração).
* `docker exec -it <container> <comando>`: executa um comando interativo dentro do container. Usamos para acessar o Redis via `redis-cli`.
* `docker compose down -v`: derruba os containers e remove os volumes associados (limpa completamente o ambiente).

Além disso, foi utilizado o AOF (Append Only File), que é uma persistência que registra todas as operações de gravação recebidas pelo servidor. Essas operações podem ser reproduzidas novamente na inicialização do servidor, reconstruindo o conjunto de dados original. Os comandos são registrados usando o mesmo formato que o próprio protocolo Redis.

## 3. Instruções de Instalação do Docker e Docker Compose (Ubuntu 22.04 / WSL2)

Neste ambiente WSL2, o Docker foi instalado via **snap** e a extensão WSL2 foi configurada manualmente. Segue o histórico de comandos utilizados:

```bash
# Atualizar e verificar WSL2
sudo apt-get update
sudo apt-get upgrade
wsl -l -v                   # lista distribuições e versões
wsl --version               # verifica versão do WSL

# Caso precise, instalar o WSL2 manualmente
sudo apt install wsl
wsl --version

# Instalar Docker via snap
sudo snap install docker   # instala Docker Engine usando Snap (um sistema de pacotes universal do Ubuntu, que gerencia versões e dependências isoladas em contêineres leves)

# Verificar instalação e versões
docker --version           # ex.: Docker version XX.X.X
docker compose version     # ex.: Docker Compose version v2.X.X

# Execução de um container de teste
docker run hello-world
```

## 4. Arquivo `docker-compose.yml` 

Crie um diretório `redis-docker/` e, dentro, o arquivo `docker-compose.yml`:

```yaml
services:
  redis:                      # 2. Definição do serviço "redis"
    image: redis:latest       # 3. Imagem Docker (tag latest)
    container_name: redis01   # 4. Nome do container
    ports:
      - "6379:6379"          # 5. Mapeia porta TCP do host para o container
    volumes:
      - redis-data:/data       # 6. Volume nomeado para persistência em /data
    restart: unless-stopped    # 7. Política de reinício automática
    command:                  # 8. Comando customizado para habilitar AOF
      - redis-server
      - --appendonly        # ativa o modo AOF (Append Only File), que grava todas as operações de escrita em disco
      - yes                # garante persistência mesmo após queda ou reinício do container

volumes:
  redis-data:                 # 9. Declaração do volume nomeado
    driver: local             #    driver local padrão
```

## 5. Comandos Utilizados para Execução e Testes

Os seguintes comandos do Docker e Docker Compose foram utilizados e compreendidos durante a execução do projeto:

```bash
cd redis-docker
# Inicia o container em background
docker compose up -d

# Lista containers ativos
docker ps

# Teste de conexão via CLI
docker exec -it redis01 redis-cli ping    # deve retornar PONG

# Teste de SET e GET
docker exec -it redis01 redis-cli SET usuario:100 Joao
docker exec -it redis01 redis-cli GET usuario:100    # deve retornar "Joao"

# Verifica diretório de dados para persistência
docker exec -it redis01 ls -lh /data/appendonlydir

# Teste de Hashes
docker exec -it redis01 redis-cli HSET bike:2 model Deimos brand Ergonom type "Enduro bikes" price 4972

# Consultas ao Hash
docker exec -it redis01 redis-cli HGETALL bike:2
docker exec -it redis01 redis-cli HGET bike:2 price

# Log
docker-compose logs -f redis

# Para parar e remover
docker compose down 
```

## 6. Prints do Serviço Funcionando

![Print do Redis em funcionamento](redis.jpeg)
![Print do Redis Hash em funcionamento](redis_hash.jpeg)
![Print do Redis AOF em funcionamento](redis_AOF.png)
![Print do Redis LOGS em funcionamento](logs.jpeg)

## 7. Dificuldades Encontradas e Soluções

* **Nome do arquivo Compose**: inicialmente uso de underscore (`docker_compose.yml`) em vez de hífen; corrigido para `docker-compose.yml`.
* **Criação de AOF**: O Redis gravava os dados no diretório `appendonlydir`, conforme identificado via `CONFIG GET dir` e `CONFIG GET appendfilename`. Para forçar a criação (ou reescrita) do arquivo de log AOF, foi utilizado o comando `BGREWRITEAOF`, assegurando a persistência dos dados mesmo após reinicialização. Além disso, durante os estudos, observamos que este erro foi causado devido a confusão nos testes, depois de realizar os testes em outro computador percebemos que não havia necessidade da utilização deste comando, porém ele é um comando muito útil.
* **Compreender como funciona**: Dúvidas em relação ao funcionamento do redis e docker, retiradas por meio de pesquisas.

## 8. Referências Utilizadas

* Docker Compose reference: [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
* Redis persistence: [https://redis.io/topics/persistence](https://redis.io/topics/persistence)
* Kinsta tutorial: [https://kinsta.com/pt/blog/executar-redis-no-docker/]
* IBM: [https://www.ibm.com/br-pt/think/topics/redis]

