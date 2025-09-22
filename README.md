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

## Criando a base de dados

Acesse o container do ctr-mysql e crie a base de dados para o Zabbix:

```bash
docker exec -it ctr-mysql mysql -u root -p
```

```sql
CREATE DATABASE IF NOT EXISTS zabbix CHARACTER SET utf8 COLLATE utf8_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '&kW97YDmywJ&13';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
exit
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
docker logs -f ctr-zbx
```

Qualquer dúvida ou sugestão, abra uma issue ou envie uma contribuição!
