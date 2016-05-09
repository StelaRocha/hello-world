# hello-world

#include "DHT.h"


DHT dht(DHTPIN, DHTTYPE);

#include <Ethernet.h>
#include <SD.h>
#include <SPI.h>

byte mac[] = {0x90, 0xA2, 0xDA, 0x00, 0x9B, 0x36}; //physical mac address
IPAddress ip(172, 16, 0, 90);
EthernetServer server(80); //server port

File webFile;

#define DHTPIN A1 // pino que estamos conectado
#define DHTTYPE DHT11 // DHT 11

#define REQ_BUF_SZ 40
char HTTP_req[REQ_BUF_SZ] = {0};
char req_index = 0;

int sensor = 0;      //Pino analógico em que o sensor está conectado.
int valorSensor = 0; //Usada para ler o valor do sensor em tempo real.
int lerDados;

const int carga1 = 9;
int flag1 = 0;

void setup() {
  pinMode(carga1, OUTPUT);
  analogReference(INTERNAL);

  Ethernet.begin(mac, ip);
  server.begin();

  Serial.begin(9600);

  Serial.println("Inicializando cartão microSD...");
  if (!SD.begin(4)) {
    Serial.println("ERRO = inicializacao do cartao falhou!");
    return;

  }
  Serial.println("SUCESSO = cartao MicroSD inicializado");

  if (!SD.exists("index.htm")) {
    Serial.println("ERRO = index.htm nao foi encontrado!");
    return;
  }
  Serial.println("SUCESSO = Encontrado o arquivo index.htm!");
}

void loop() {
  EthernetClient client = server.available();

  if (client) {
    boolean currentLineIsBlank = true;
    while (client.connected()) {
      if (client.available()) {

        char c = client.read();

        if (req_index < (REQ_BUF_SZ - 1)) {

          HTTP_req[req_index] = c;
          req_index++;
        }

        if (c == '\n' && currentLineIsBlank) {
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/html");
          client.println("Connection: close");
          client.println();

          if (StrContains(HTTP_req, "ajax_LerDados")) {
            LerDados (client);
          }

          if (StrContains(HTTP_req, "ajax_carga1")) {
            SetCarga1();
          }

          else {

            webFile = SD.open("index.htm");
            if (webFile) {
              while (webFile.available()) {
                client.write(webFile.read());
              }
              webFile.close();
            }
          }
          Serial.println(HTTP_req);
          req_index = 0;
          StrClear(HTTP_req, REQ_BUZ_SZ);
          break;
        }
        if (c == '\n') {
          currentLineIsBlank = true;

        }
        else if (c != '\r') {
          currentLineIsBlank = false;

        }
      }
    }
    delay(1);
    client.stop();
  }

}

void LerDados(EthernetClient novoClient) {
  valorSensor = analogRead(sensor);
  valorSensor = map(valorSensor, 0, 1023, 0, 100);
  novoClient.print(valorSensor);
  novoClient.println("%");

  novoClient.print("|");

  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(t) || isnan(h))
  {
    novoClient.println("Failed to read from DHT");
  }
  else
  {
    novoClient.print("Umidade: ");
    novoClient.print(h);
    novoClient.println(" %");
    novoClient.print("Temperatura: ");
    novoClient.print(t);
    novoClient.println(" *C");
    novoClient.println();

    novoCliente.print("|");

    novoCliente.print(flag1);

    novoCliente.print("|");

    delay(500);
  }

}
void StrClear(char *str, char length) {
  for (int i = o; i < length; i++) {
    str[1] = 0;
  }
}


char StrContains(char *str, char *sfind) {

  char found = 0;
  char index = 0;
  char len;

  len = strlen(str);

  if (strlen(sfind) > len) {
    return 0;

  }
  while (index < len) {
    if (str[index] == sfind[found]) {
      found++;
      if (strlen(sfind) == found) {
        return 1;
      }
    }
    else {
      found = 0;
    }
    index++;
  }
  return 0;
}

void SetCarga1(){
  if(flag1 == 0){
    digitalWrite(carga1, HIGH);
    flag1 = 1;
    
  }
  else{
    digitalWrite(carga1, LOW);
    flag1 = 0;
  }
  
}

