#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT22

#define ZPH_RX 16

DHT dht(DHTPIN, DHTTYPE);

HardwareSerial ZPH(2);

byte data[9];

void setup() {

  Serial.begin(115200);

  dht.begin();

  ZPH.begin(9600, SERIAL_8N1, ZPH_RX, -1);

  Serial.println("Smart Hospital Air Guard");
  Serial.println("System Starting...");

}

void loop() {

  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  Serial.println("========================");

  if (!isnan(temperature) && !isnan(humidity)) {

    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.println(" C");

    Serial.print("Humidity: ");
    Serial.print(humidity);
    Serial.println(" %");

  } else {

    Serial.println("DHT22 Error");

  }

  if (readZPH()) {

    float pmRate = data[3] + data[4] / 100.0;

    byte voc = data[7];

    Serial.print("PM2.5 Low Pulse Rate: ");
    Serial.print(pmRate);
    Serial.println(" %");

    Serial.print("VOC Grade: ");
    Serial.println(voc);

  } else {

    Serial.println("Waiting for ZPH02...");

  }

  delay(2000);

}

bool readZPH() {

  while (ZPH.available() >= 9) {

    if (ZPH.read() == 0xFF) {

      data[0] = 0xFF;

      for (int i = 1; i < 9; i++) {
        data[i] = ZPH.read();
      }

      byte checksum = 0;

      for (int i = 1; i < 8; i++) {
        checksum += data[i];
      }

      checksum = (~checksum) + 1;

      if (data[1] == 0x18 &&
          data[8] == checksum) {

        return true;

      }

    }

  }

  return false;

}
