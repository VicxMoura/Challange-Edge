# IoT Sensor Ultrassônico para Medição de Velocidade

# Integrantes
Victoria Moura - RM555474
Nickolas Moreno Cardoso - RM557132
Andre Giovane De Maria - RM556384
Kaio Drago Lima Souza - RM556095
Lucas Imparato - RM554896

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

- **Wokwi** (para programar o ESP32)
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
   - Abra o código no Wokwi e ajuste as configurações de rede (SSID e senha) e o broker MQTT.
   - Faça o upload do código no ESP32.

2. **Configuração do Node-RED**
   - Importe o fluxo fornecido no repositório para o Node-RED.
   - Verifique se o Node-RED está conectado ao HiveMQ e recebendo os dados do ESP32.

3. **Monitoramento**
   - Acesse o dashboard no Node-RED para monitorar a velocidade em tempo real.

## Código Fonte

#include <WiFi.h>
#include <PubSubClient.h>

// Definir o SSID e a senha da rede Wi-Fi (substitua pelos seus valores)
const char* ssid = "Wokwi-GUEST";  // Rede padrão no Wokwi
const char* password = "";         // Sem senha para a rede Wokwi-GUEST

// Usar o broker público HiveMQ
const char* mqtt_server = "broker.hivemq.com";  // Broker MQTT HiveMQ

WiFiClient espClient;
PubSubClient client(espClient);

// Definir pinos do sensor ultrassônico
#define TRIG_PIN 5
#define ECHO_PIN 18

// Função para medir a distância usando o sensor ultrassônico
long measureDistance() {
  // Enviar o pulso do trigger
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Ler o tempo do echo
  long duration = pulseIn(ECHO_PIN, HIGH);
  
  // Calcular a distância em centímetros
  long distance = (duration * 0.0343) / 2;

  // Limitar a distância para evitar ruídos (0 cm ou > 400 cm são valores suspeitos para esse sensor)
  if (distance <= 0 || distance > 400) {
    return -1; // Retorna -1 para indicar erro de leitura
  }

  return distance;
}

void setup() {
  Serial.begin(115200);
  
  // Configurar os pinos do sensor ultrassônico
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  // Conectar ao Wi-Fi
  Serial.println("Conectando ao Wi-Fi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando...");
  }
  Serial.println("Conectado ao Wi-Fi!");

  // Configurar o servidor MQTT
  client.setServer(mqtt_server, 1883);
}

// Função para reconectar ao broker MQTT, se necessário
void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentando conectar ao broker MQTT...");
    if (client.connect("ESP32Client")) {
      Serial.println("Conectado ao broker MQTT");
    } else {
      Serial.print("Falhou, rc=");
      Serial.println(client.state());
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Medir a distância em dois intervalos de tempo
  long distance1 = measureDistance();
  delay(100);  // Espera 100ms
  long distance2 = measureDistance();

  // Verificar se as leituras são válidas
  if (distance1 == -1 || distance2 == -1) {
    Serial.println("Erro na leitura da distância");
    return;
  }

  // Calcular a variação de distância e a velocidade (em cm/s)
  long deltaDistance = distance2 - distance1;
  float velocity = deltaDistance / 0.1;  // Velocidade em cm/s (0.1 segundos)

  // Criar a string da mensagem para enviar via MQTT
  String payload = "Velocidade: " + String(velocity) + " cm/s";
  Serial.print("Publicando: ");
  Serial.println(payload);

  // Publicar os dados no tópico "iot/velocidade"
  if (client.publish("iot/velocidade", payload.c_str())) {
    Serial.println("Mensagem publicada com sucesso!");
  } else {
    Serial.println("Falha ao publicar mensagem.");
  }

  // Esperar 2 segundos antes de repetir a medição
  delay(2000);
}

## Dependências

- **Wokwi**
- **Node-RED**
- **Broker MQTT HiveMQ**
  
## Conclusão

Esta solução IoT oferece uma maneira eficiente de monitorar a velocidade dos carros em tempo real, utilizando um sensor ultrassônico e uma arquitetura leve e escalável, com integração fácil com o Node-RED e o HiveMQ.