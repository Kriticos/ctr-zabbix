# Projeto Zabbix com Docker Compose

Este repositório fornece um ambiente Docker Compose para provisionar os containers Zabbix Server, Frontend e Agent, todos configurados via variáveis de ambiente.

## Pré-requisitos

- Docker e Docker Compose.
- Rede Docker `network-share` já criada:
- Container ctr-mysql rodando.

## Criar a rede externa se ainda não existir

```bash
docker network create --driver bridge network-share --subnet=172.18.0.0/16
```

### OBSERVAÇÃO

**Ajuste a subnet conforme necessário.**

## Estrutura de arquivos

```plaintext
bskp/
├── scripts/                 # Scripts personalizados (se necessário)
└── ctr-zbx/                 # Projeto Zabbix
     ├── docker-compose.yml  # Definição dos serviços Docker
     ├── .env.example        # Exemplo de variáveis de ambiente
     └── README.md           # Documentação do serviço
```

## Permissões das Pastas

```bash
chown -R root:root /bskp/ctr-zbx
chmod -R 750 /bskp/ctr-zbx
```

## Configuração das variáveis de ambiente

Copie o template e preencha os valores:

```bash
cp /bskp/ctr-zbx/.env.example /bskp/ctr-zbx/.env
```

Ajuste as variáveis no arquivo `.env`.

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
