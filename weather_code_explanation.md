Kod naszej aplikacji został napisany w języku C++. Pełna dokumentacja dostępnych funkcji dostępna jest tutaj:
https://www.arduino.cc/reference/en/

Import - czyli wczytanie dodatkowych bibliotek (dodatkowego kodu, dodatkowych programów), z których będzie korzystał nasz program. 
W naszym przykładzie są to biblioteki:

* ESP8266Wifi - do obsługi modułu NodeMCU
* BlynkSimpleEsp8266 - do komunikacji z aplikacją Blynk
* DHT - do obsługi modułu DHT11


```
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>
```

Poniższa linijka służy do podania tokena, czyli kod który 'łączy' nasz projekt w aplikacji Blynk z naszym programem tutaj.

```
char auth[] = "Blynk_auth_token";
```

Poniższe dwie linijki służą do podania nazwy naszej domowej sieci WIFI (pole SSID) oraz hasła (pole pass - z angielskiego Password to hasło). UWAGA: wielkość liter ma znaczenie!
```
char ssid[] = "Nazwa_sieci_(SSID)";
char pass[] = "Haslo";
```

Poniższa linijka służy do powiedzenia programowi, żeby wszystkie komunikaty BLYNK wypisywał używając Monitor Portu Szeregowego.

```
#define BLYNK_PRINT Serial
```

Poniższe 2 linijki spowodują, że wszędzie tam gdzie podamy DHTPIN aplikacja podstawi sobie wartość 4. Natomiast w miejsce DHTTYPE wstawi DHT11

```
#define DHTPIN 4          // D2
#define DHTTYPE DHT11     // DHT 11
```

Poniższy kod służy do konfiguracji biblioteki dla czujnika DHT. Informujemy nim, że czujnik DHT jest podpięty do pinu oznaczonego jako D2 na płytce Node MCU, a typ czujnika to DHT11.

```
DHT dht(DHTPIN, DHTTYPE);
```

Poniższa linijka tworzy nam zmienną globalną o nazwie sensorTimer - dzięki temu będziemy mogli używać sensorTimera w wielu miejscach. Timer jest czymś w rodzaju zegara, który będzie odpowiedzialny za uruchamianie naszego kodu co pewien czas.

```
BlynkTimer sensorTimer;
```

Poniższa funkcja jest sercem naszej aplikacji - to tutaj odbywają się główne operacje. Co sekundę pobierane są dane z czujnika temperatury, następnie z czujnika wilgotności, jeśli któraś z liczb nie jest liczbą (sprawdzanie: isnan(temperature) oraz isnan(humidity)) to wtedy wypisujemy na Monitor Portu Szeregowego "Zly odczyt z sensora DHT" i konczymy. Natomiast jestli oba odczyty z czujników są dobre to wypisujemy na Monitor Portu Szeregowego tekst "Dobry odczyt" i wysyłamy do aplikacji Blynk wartość temperatury (na wirtualny port V1) oraz wilgotności (na wirtualny port V2).

```
void sendSensorData() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Zly odczyt z sensora DHT.");
    return;
  } else {
    Serial.println("Dobry odczyt.");
  }

  Blynk.virtualWrite(V1, temperature);
  Blynk.virtualWrite(V2, humidity);
}
```

Poniższa funkcja setup() odpowiada za konfigurację początkową naszego programu - ustawiamy tutaj szybkość połączenia z jaką będziemy wyświetlać tekst na Monitorze Portu Szeregowego. Uruchamia połączenie z WiFi. Uruchamia czujnik DHT oraz ustawia czas co jaki dane z czujnika DHT są zczytywane (1000L oznacza 1000 milisekund - czyli 1 sekundę).

```
void setup() {
  Serial.begin(115200);

  Blynk.begin(auth, ssid, pass);

  dht.begin();

  sensorTimer.setInterval(1000L, sendSensorData);
}
```

Poniżej znajduje się funkcja, która jest uruchamiana przez Node MCU cały czas w kółko.

```
void loop() {
  Blynk.run();
  sensorTimer.run();
}
```

**Kod aplikacji piszemy tylko w języku angielskim. Ważne jest by uczyć się języka bo cała aktualna dokumentacja i wszelkie fora są również w języku angielskim.**

