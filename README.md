# Projeto Zabbix com Docker Compose

Este repositório fornece um ambiente Docker Compose para provisionar os containers Zabbix Server, Frontend e Agent, todos configurados via variáveis de ambiente.

## Pré-requisitos

- Docker e Docker Compose instalados na máquina.  
- Rede Docker externa chamada `network-share` já criada:
- Container MSYQL rodando.

```bash
# Criar a rede externa se ainda não existir
docker network create --driver bridge --subnet 172.18.0.0/16 network-share
```

- Copiar o exemplo de variáveis para um arquivo `.env`:

```bash
cp .env.example .env
```

## Estrutura de arquivos

```plaintext
├── docker-compose.yml    # Definição dos serviços Docker
├── .env.example          # Exemplo de variáveis de ambiente
└── README.md             # Este arquivo de documentação
```

## Permissões das Pastas

```bash
chown -R root:root /bskp/zabbix
chmod -R 750 /bskp/zabbix
```

## Configuração das variáveis de ambiente

No arquivo `.env`, defina:

```bash
# Nomes dos containers
ZBX_SERVER_HOST=<NOME_CONTAINER_SERVER>
ZBX_SERVER_FRONTEND=<NOME_CONTAINER_FRONTEND>
ZBX_SERVER_AGENT=<NOME_CONTAINER_AGENT>

# Versão das imagens Zabbix
RELEASE=ubuntu-7.2-latest

# Banco de dados MySQL
DB_SERVER_HOST=<NOME_CONTAINER_MYSQL>
MYSQL_DATABASE=zabbix
MYSQL_USER=zabbix
MYSQL_PASSWORD=<SENHA_ZABBIX>
MYSQL_ROOT_PASSWORD=<SENHA_ROOT_MYSQL>

# Time e timezone
VOL_LOCALTIME=/etc/localtime:/etc/localtime:ro
VOL_TZ=/etc/timezone:/etc/timezone:ro
PHP_TZ=America/Sao_Paulo

# Portas (host:container)
PORTS_SRV_ZBX=10051:10051
PORTS_SRV_FRONTEND=<PORTA_HOST>:8080
PORTS_SRV_AGENT=10050:10050

# Rede estática
IPV4_ZBX_SERVER_HOST=<IP_SERVER>
IPV4_ZBX_FRONTEND=<IP_FRONTEND>
IPV4_ZBX_AGENT=<IP_AGENT>
SUBNET=<SUBNET>
```

## Subindo o serviço

Para iniciar todos os containers em segundo plano:

```bash
docker compose up -d
```

Verifique o status:

```bash
docker ps | grep zbx
```

Acompanhe os logs do Server:

```bash
docker logs -f ${ZBX_SERVER_HOST}
```

## Parando e removendo containers

Para parar e remover todos os recursos:

```bash
docker compose down
```

## Explicação do `docker-compose.yml`

```yaml
services:
  zbx-server:
    image: zabbix/zabbix-server-mysql:${RELEASE}
    container_name: ${ZBX_SERVER_HOST}
    environment:
      - DB_SERVER_HOST=${DB_SERVER_HOST}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    ports:
      - ${PORTS_SRV_ZBX}
    volumes:
      - ${VOL_LOCALTIME}
      - ${VOL_TZ}
    networks:
      network-share:
        ipv4_address: ${IPV4_ZBX_SERVER_HOST}
    restart: unless-stopped

  zbx-frontend:
    image: zabbix/zabbix-web-apache-mysql:${RELEASE}
    container_name: ${ZBX_SERVER_FRONTEND}
    environment:
      - ZBX_SERVER_HOST=${ZBX_SERVER_HOST}
      - DB_SERVER_HOST=${DB_SERVER_HOST}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - PHP_TZ=${PHP_TZ}
    ports:
      - ${PORTS_SRV_FRONTEND}
    volumes:
      - ${VOL_LOCALTIME}
      - ${VOL_TZ}
    networks:
      network-share:
        ipv4_address: ${IPV4_ZBX_FRONTEND}
    restart: unless-stopped

  zbx-agent:
    image: zabbix/zabbix-agent2:${RELEASE}
    container_name: ${ZBX_SERVER_AGENT}
    build:
      context: /sistemas/
      dockerfile: docker/dockerfile.zbx-agent
    environment:
      - ZBX_HOSTNAME=${ZBX_SERVER_HOST}
      - ZBX_SERVER_HOST=${ZBX_SERVER_HOST}
    ports:
      - ${PORTS_SRV_AGENT}
    volumes:
      - ${VOL_LOCALTIME}
      - ${VOL_TZ}
      - ${VOL_SCRIPTS_PATH}
    networks:
      network-share:
        ipv4_address: ${IPV4_ZBX_AGENT}
    restart: unless-stopped

networks:
  network-share:
    external: true
    ipam:
      config:
        - subnet: ${SUBNET}
```

## Observações

- **Volumes de timezone**: garantem que o container use o mesmo horário do host.  
- **Rede estática**: cada serviço recebe um IP fixo na sub-rede definida.  
- **Agent customizado**: ajuste `context` e `dockerfile` conforme sua estrutura de diretórios.  
- **Reinício automático**: `restart: unless-stopped` mantém o serviço ativo após quedas ou reinicializações.  

---

Qualquer dúvida ou sugestão, abra uma issue ou envie uma contribuição!
