/*
  LoRa Client w/ sensor for Arduino :
  Support Devices: LoRa Shield + Arduino + DHT11 + Sensor Ultrassonico + Sensor de Chama 
  Support Devices: LoRa Shield + Arduino + DHT11 + Sensor Ultrassonico + Sensor de Chama + Sensor de Luminosidade
  
  Example sketch showing how to create a messageing client, 
  with the RH_RF95 class. RH_RF95 class does not provide for addressing or
  reliability, so you should only use RH_RF95 if you do not need the higher
  level messaging abilities.
  It is designed to work with the other example LoRa Simple Server
  User need to use the modified RadioHead library from:
  https://github.com/dragino/RadioHead
  modified 16 11 2016
  by Edwin Chen <support@dragino.com>
  Dragino Technology Co., Limited
*/
#include <SPI.h>
#include <RH_RF95.h>
//Cte sensor distancia
#define echoPin 3 // attach pin D2 Arduino to pin Echo of HC-SR04
#define trigPin 4 //attach pin D3 Arduino to pin Trig of HC-SR04
//Cte temperatura e umidade
#include "DHT.h"
#define DHTPIN  A0 //pino utilizado no esp
#define DHTTYPE DHT11   // DHT 11
//Cte sensor Luminosidade
#define LIGHTPIN A2

// defines variaveis sensor distancia
long duration; // variable for the duration of sound wave travel
int distance; // variable for the distance measurement
//variaveis sensor chama
int pino_D0 = 5;
int pino_A0 = A1;
int valor_a = 0;
int valor_d = 0;
int chama = 0;
float temp,humid;
char tem_1[8]={"\0"},hum_1[8]={"\0"},dis_1[8]={"\0"},cha_1[4]={"\0"};
char *node_id = "<12345>"; //From LG01 via web Local Channel settings on MQTT.Please refer <> dataformat in here.
int deviceId = 111;
float temp,humid,luz;
char tem_1[8]={"\0"},hum_1[8]={"\0"},luz_1[8]={"\0"},dis_1[8]={"\0"},cha_1[4]={"\0"},deviceId_1[6] = {"\0"};
char *node_id = "<16a>"; //From LG01 via web Local Channel settings on MQTT.Please refer <> dataformat in here.
uint8_t datasend[70];
unsigned int count = 1;

DHT dht(DHTPIN, DHTTYPE); //Inicia Biblioteca
// Singleton instance of the radio driver
RH_RF95 rf95;
//The parameter are pre-set for 868Mhz used. If user want to use lower frenqucy 433Mhz.Better to set 
//rf95.setSignalBandwidth(31250);
//rf95.setCodingRate4(8);
float frequency = 915.0;
void setup() 
{
  Serial.begin(9600);
  //while (!Serial) ; // Wait for serial port to be available
  pinMode(pino_A0, INPUT);
  pinMode(pino_D0, INPUT);
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an OUTPUT
  pinMode(echoPin, INPUT); // Sets the echoPin as an INPUT
  dht.begin();
  Serial.println("Start LoRa Client");
  if (!rf95.init())
    Serial.println("init failed");
  // Setup ISM frequency
  rf95.setFrequency(frequency);
  // Setup Power,dBm
  rf95.setTxPower(13);
  // Setup Spreading Factor (6 ~ 12)
   rf95.setSpreadingFactor(7);
  // Setup BandWidth, option: 7800,10400,15600,20800,31250,41700,62500,125000,250000,500000
  //Lower BandWidth for longer distance.
  rf95.setSignalBandwidth(125000);
  // Setup Coding Rate:5(4/5),6(4/6),7(4/7),8(4/8) 
  rf95.setCodingRate4(5);
  rf95.setSyncWord(0x34);
  /*
  //Different Combination for distance and speed examples: 
  Example 1: Bw = 125 kHz, Cr = 4/5, Sf = 128chips/symbol, CRC on. Default medium range
    rf95.setSignalBandwidth(125000);
    rf95.setCodingRate4(5);
    rf95.setSpreadingFactor(7);
  Example 2: Bw = 500 kHz, Cr = 4/5, Sf = 128chips/symbol, CRC on. Fast+short range
    rf95.setSignalBandwidth(500000);
    rf95.setCodingRate4(5);
    rf95.setSpreadingFactor(7);
  Example 3: Bw = 31.25 kHz, Cr = 4/8, Sf = 512chips/symbol, CRC on. Slow+long range
    rf95.setSignalBandwidth(31250);
    rf95.setCodingRate4(8);
    rf95.setSpreadingFactor(9);
  Example 4: Bw = 125 kHz, Cr = 4/8, Sf = 4096chips/symbol, CRC on. Slow+long range
    rf95.setSignalBandwidth(125000);
    rf95.setCodingRate4(8);
    rf95.setSpreadingFactor(12); 
  */
}
void dhtTempHumid()
{
  temp = dht.readTemperature();
  humid = dht.readHumidity();
  Serial.println(F("Temperatura e Humidade:"));
  Serial.print("[");
  Serial.print(temp);
  Serial.print("℃");
  Serial.print(",");
  Serial.print(humid);
  Serial.print("%");
  Serial.print("]");
  Serial.println("");
  delay(3000);
}
void Ultrassom()
{
  // Clears the trigPin condition
  digitalWrite(trigPin, LOW);
  delayMicroseconds(3); //delay entre digitalWrite
  // Sets the trigPin HIGH (ACTIVE) for 10 microseconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(15); //delay entre digitalWrite
  digitalWrite(trigPin, LOW);
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
  // Calculating the distance
  distance = duration * 0.034 / (2*10); // Speed of sound wave divided by 2 (go and back)
  distance = duration * 0.034 / 2; // Speed of sound wave divided by 2 (go and back)
  // Displays the distance on the Serial Monitor
  Serial.print("Distance: ");
  Serial.print(distance); //Valor em centimetros que iremos enviar para a Dojot
  //Serial.println(" cm");    
  delay(500);
  delay(1000);
}

void is_chama()
{
  int valor_a = analogRead(pino_A0);
  int valor_d = digitalRead(pino_D0);
  Serial.println(F("valor chama: "));
  Serial.print(valor_d);
  Serial.print(".");
  if (valor_d != 1)
  {
    Serial.println("Fogo detectado !!!");
  }else{
    Serial.println("Nada identificado!!!");
  }
  delay(500);
}

void valor_luz(){
  luz = analogRead(LIGHTPIN);
  // Serial.println(F("valor luz: "));
  // Serial.print(luz);
  delay(500);
}

void sensorWrite()
{
    char data[70] = "\0";
    for(int i = 0; i < 70;i++)
    {
       data[i] = node_id[i];
    }
    dtostrf(temp,0,1,tem_1);
    dtostrf(humid,0,1,hum_1);
    dtostrf(distance,0,1,dis_1);
    dtostrf(luz,0,1,luz_1);
    //dtostrf(valor_d,0,1,cha_1);
    itoa(valor_d,cha_1,4);
    //tmp, umd, xma, rssi,sonic
    Serial.println("debugInicial:");
    Serial.println(tem_1);
    Serial.println(hum_1);
    Serial.println(dis_1);
    Serial.println(cha_1);
     strcat(data,"{\"tmp\":");
    itoa(valor_d,cha_1,10);
    itoa(deviceId,deviceId_1,10);
    /*tmp, umd, xma, sonic
     * pd -> payload
    * i -> id da dojot
    * t -> temperatura
    * u -> umidade
    * x -> chama
    * s -> ultrassom
    * l -> luminosidade
    */
    //Serial.println("debugInicial:");
    //Serial.println(tem_1);
    //Serial.println(hum_1);
    //Serial.println(dis_1);
    //Serial.println(cha_1);
    //Serial.println(luz_1);
     strcat(data,"{\"pld\":");
     strcat(data,"{\"i\":");
     strcat(data,deviceId_1);
     strcat(data,",\"t\":");
     strcat(data,tem_1);
     strcat(data,",\"umd\":");
     strcat(data,",\"u\":");
     strcat(data,hum_1);
     strcat(data,",\"xma\":");
     strcat(data,",\"x\":");
     strcat(data,cha_1);
     strcat(data,",\"sonic\":");
     strcat(data,",\"s\":");
     strcat(data,dis_1);
     strcat(data,"}");
     strcat(data,",\"l\":");
     strcat(data,luz_1);
     strcat(data,"}}");
     strcpy((char *)datasend,data);

   Serial.println("debug pacote: ");
   Serial.println((char *)datasend);
   Serial.println(sizeof datasend);     
}
void SendData()
{
  Serial.println(F("Sending data to LG01"));
           
   
      rf95.send((char *)datasend,sizeof(datasend));  
      rf95.waitPacketSent();  // Now wait for a reply
    
      uint8_t buf[RH_RF95_MAX_MESSAGE_LEN];
      uint8_t len = sizeof(buf);
     if (rf95.waitAvailableTimeout(3000))
  { 
    // Should be a reply message for us now   
    if (rf95.recv(buf, &len))
   {
     
      Serial.print("got reply from LG01: ");
      Serial.println((char*)buf);
      Serial.print("RSSI: ");
      Serial.println(rf95.lastRssi(), DEC);    
    }
    else
    {
      Serial.println("recv failed");
    }
  }
  else
  {
    Serial.println("No reply, is LoRa server running?");
  }
  delay(5000);
}
void loop()
{ 
  Serial.print("###########    ");
  Serial.print("COUNT=");
  Serial.print(count);
  Serial.println("    ###########");
  count++;
  dhtTempHumid();
  is_chama();
  Ultrassom();
  valor_luz();
  sensorWrite();
  SendData();
}
