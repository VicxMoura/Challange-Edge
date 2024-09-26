# IoT Sensor Ultrassônico para Medição de Velocidade

## Descrição do Projeto

Este projeto consiste no desenvolvimento de uma solução IoT que utiliza o sensor ultrassônico HC-SR04, acoplado a um ESP32, para medir a velocidade de um objeto em movimento. Os dados de velocidade são transmitidos para o HiveMQ via protocolo MQTT e exibidos em tempo real no Node-RED, proporcionando uma interface simples e interativa para monitorar a velocidade.

### Arquitetura do Sistema

A arquitetura do projeto é composta pelos seguintes componentes:

1. **Dispositivo IoT**:
   - **ESP32**: Microcontrolador responsável por coletar os dados do sensor ultrassônico e enviar as leituras para o broker MQTT.
   - **Sensor Ultrassônico (HC-SR04)**: Sensor responsável por medir a distância entre o sensor e o objeto, permitindo calcular a variação de distância e, com isso, a velocidade.

2. **Back-end**:
   - **Broker MQTT (HiveMQ)**: Responsável por receber os dados do ESP32 e distribuí-los para os clientes MQTT, como o Node-RED.
   
3. **Front-end**:
   - **Node-RED**: Plataforma visual de desenvolvimento que recebe os dados do broker MQTT e exibe a velocidade em tempo real em um dashboard interativo.

### Diagrama da Arquitetura

```plaintext
+---------------+       +-------------+      +-----------------+
|  ESP32        |  ---> |  HiveMQ      | ---> | Node-RED         |
| (Sensor       |       |  (Broker     |      | (Dashboard)      |
|  Ultrassônico) |       |  MQTT)       |      |                 |
+---------------+       +-------------+      +-----------------+
```

## Recursos Necessários

### Hardware

- **ESP32**
- **Sensor Ultrassônico HC-SR04**
- **Cabos de conexão**
- **Fonte de alimentação USB**

### Software

- **Arduino IDE** ou **PlatformIO** (para programar o ESP32)
- **Node-RED** (para visualização dos dados)
- **Broker MQTT HiveMQ**
- **Bibliotecas Arduino**:
  - `WiFi.h`
  - `PubSubClient.h`

## Configurações e Instalações

### 1. Preparação do ESP32

- Conecte o sensor ultrassônico ao ESP32:
  - **VCC** do sensor ao **3.3V** do ESP32
  - **GND** ao **GND**
  - **TRIG** ao **pino D5**
  - **ECHO** ao **pino D18**

- Código para ESP32 (incluído no repositório):
  - Configure a conexão Wi-Fi e o broker MQTT.
  - Leia os dados do sensor ultrassônico e calcule a velocidade.
  - Envie as leituras de velocidade via MQTT para o broker HiveMQ.

### 2. Configuração do Broker MQTT (HiveMQ)

- Utilize o HiveMQ como broker MQTT.
- Configure o Node-RED para se conectar ao broker via o IP ou hostname e a porta MQTT (geralmente 1883).

### 3. Dashboard no Node-RED

- O Node-RED será responsável por consumir as mensagens publicadas no tópico MQTT e exibir a velocidade em tempo real.
- Utilize um gráfico ou indicador para visualizar os dados de velocidade.

## Instruções de Uso

1. **Configuração do ESP32**
   - Abra o código na Arduino IDE e ajuste as configurações de rede (SSID e senha) e o broker MQTT.
   - Faça o upload do código no ESP32.

2. **Configuração do Node-RED**
   - Importe o fluxo fornecido no repositório para o Node-RED.
   - Verifique se o Node-RED está conectado ao HiveMQ e recebendo os dados do ESP32.

3. **Monitoramento**
   - Acesse o dashboard no Node-RED para monitorar a velocidade em tempo real.

## Código Fonte



## Dependências

- **Arduino IDE**
- **Node-RED**
- **Broker MQTT HiveMQ**
  
## Conclusão

Esta solução IoT oferece uma maneira eficiente de monitorar a velocidade dos carros em tempo real, utilizando um sensor ultrassônico e uma arquitetura leve e escalável, com integração fácil com o Node-RED e o HiveMQ.