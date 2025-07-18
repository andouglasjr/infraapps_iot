# Aula 6 — InfraApp: Infraestrutura de Aplicações IoT

Este repositório contém os materiais e configurações utilizados na **Aula 6 de Programação de Soluções IoT**, focada em **Infraestrutura de Aplicações (InfraApps)** para sistemas baseados em Internet das Coisas.

## 🌐 O que são InfraApps?

Na IoT, dispositivos coletam dados do mundo físico, mas **é a InfraApp que dá vida a esses dados**: processa, analisa, armazena e apresenta em sistemas finais.

### Composição típica de uma InfraApp:

- **Broker MQTT**: Comunicação entre dispositivos e sistemas.
- **Middleware / Gateway IoT**: Processamento local, roteamento de protocolos, autenticação.
- **Banco de Dados (Time Series)**: Armazenamento eficiente com timestamps.
- **Dashboards e Aplicativos**: Interface de visualização dos dados.
- **Serviços em Nuvem**: Escalabilidade, APIs, Machine Learning.
- **Serviços de Backend**: Regras de negócio e integração.

---

## ⚙️ Stack Utilizada

Para fins didáticos, a aplicação de exemplo roda **localmente usando `docker-compose`**, com os seguintes serviços:

### 1. Mosquitto (MQTT Broker)

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto
    ports:
      - "1883:1883"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
```

📄 Arquivos necessários:

- `mosquitto.conf` (configuração do broker)
- `pwfile` (arquivo de senhas para autenticação)

📘 Referência: [sukesh-ak/setup-mosquitto-with-docker](https://github.com/sukesh-ak/setup-mosquitto-with-docker)

---

### 2. Telegraf (Middleware IoT)

O **Telegraf** atua como middleware na arquitetura IoT. Ele se conecta ao broker MQTT, consome mensagens publicadas por dispositivos e envia esses dados para o InfluxDB.

#### Configuração no `docker-compose.yml`

```yaml
telegraf:
  image: telegraf:1.30
  depends_on:
    - influxdb
    - mosquitto
  volumes:
    - ./telegraf/telegraf.conf:/etc/telegraf/telegraf.conf:ro
  environment:
    - HOST_PROC=/host/proc
    - HOST_SYS=/host/sys
```

> 💡 O caminho do volume deve apontar para o arquivo `telegraf.conf` local corretamente configurado.

#### Exemplo de `telegraf.conf`

A configuração do Telegraf deve ter as seções:

- **Agent**: define o comportamento básico
- **Input Plugin**: para consumir do MQTT
- **Output Plugin**: para enviar ao InfluxDB

##### Entrada MQTT

```toml
[[inputs.mqtt_consumer]]
  servers = ["tcp://mosquitto:1883"]
  topics = ["sensores/aht10"]
  qos = 0
  connection_timeout = "30s"
  client_id = "telegraf"
  username = "iot"
  password = "12345678"
  data_format = "json"
  tag_keys = []
```

##### Saída para InfluxDB

```toml
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "influx_token"
  organization = "iotlab"
  bucket = "sensores"
```

---

### 3. InfluxDB (Banco de Dados de Séries Temporais)

Banco de dados usado para armazenar dados sensoriais com marcação temporal.

#### Configuração no `docker-compose.yml`

```yaml
influxdb:
  image: influxdb:2.7
  ports:
    - "8086:8086"
  environment:
    - DOCKER_INFLUXDB_INIT_MODE=setup
    - DOCKER_INFLUXDB_INIT_USERNAME=admin
    - DOCKER_INFLUXDB_INIT_PASSWORD=admin123
    - DOCKER_INFLUXDB_INIT_ORG=iotlab
    - DOCKER_INFLUXDB_INIT_BUCKET=sensores
    - DOCKER_INFLUXDB_INIT_TOKEN=meu-token
  volumes:
    - influxdb-data:/var/lib/influxdb2
```

> Após inicializar o container, é possível acessar a UI do InfluxDB em `http://localhost:8086` e gerar tokens adicionais conforme necessário.

---

### 4. Grafana (Dashboard para Visualização)

O **Grafana** é usado para visualizar os dados coletados de forma gráfica e interativa.

#### Configuração no `docker-compose.yml`

```yaml
grafana:
  image: grafana/grafana:10.4.1
  container_name: grafana
  ports:
    - "3000:3000"
  volumes:
    - grafana-data:/var/lib/grafana
  environment:
    - GF_SECURITY_ADMIN_USER=admin
    - GF_SECURITY_ADMIN_PASSWORD=admin
  depends_on:
    - influxdb
```

Após o deploy, acesse: [http://localhost:3000](http://localhost:3000)  
Usuário: `admin`  
Senha: `admin`

> Configure o InfluxDB como fonte de dados no painel do Grafana para começar a criar gráficos com os dados da aplicação.

---

## 📊 Resultado Esperado

Com a stack em execução:

- Dispositivos enviam dados via MQTT.
- Telegraf consome e envia os dados ao InfluxDB.
- Grafana apresenta os dados de forma visual e dinâmica.

---

## 🚀 Como Rodar

1. Clone o repositório:
   ```bash
   git clone https://github.com/seu-usuario/seu-repo.git
   cd seu-repo
   ```

2. Configure os arquivos:
   - `mosquitto.conf` e `pwfile` dentro de `./mosquitto/config/`
   - `telegraf.conf` no caminho correto

3. Suba os containers:
   ```bash
   docker-compose up -d
   ```

4. Acesse:
   - Grafana: [http://localhost:3000](http://localhost:3000)
   - InfluxDB: [http://localhost:8086](http://localhost:8086)

---

## 🧠 Créditos

Material didático elaborado por **Andouglas G Silva Júnior** para a disciplina de **Programação de Soluções IoT**.
