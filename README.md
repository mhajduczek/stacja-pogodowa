# README #

Projekt stacji pogodowej z Node MCU & DHT11 & Blynk. Jest to świetny sposób do nauki programowania dla najmłodszych i nie tylko.

### Części systemu ###

* Kabel microUSB + zasilacz 5V (może być z telefonu, ważne by dostarczał prąd o natężeniu co najmniej 1A)
* Płytka Node MCU
* Czujnik Temperatury i wilgotności DHT11
* Aplikacja Blynk pokazująca odczyty z czujnika temperatury i wilgotności


### Instalacja i konfiguracja aplikacji Blynk ###

* Pobierz aplikację na Twój telefon ze sklepu Google Play lub App Store.
* Zarejestruj się w aplikacji.
* Utwórz nowy projekt. Podaj jego nazwę (Project name), wybierz z urządzeń (Choose device) ESP8266, następnie jako typ połączenia (Connection type) wybierz WiFi i kliknij w przycisk Create Project.

<img src="images/nowy_projekt.png" height=450px></img>

Po kliknięciu w przycisk Create Project pokaże się komunikat informujący o tym, że Auth Token został wysłany na email użyty podczas rejestracji:

<img src="images/projekt_utworzony.png" height=450px></img>

Jest to bardzo ważny krok - Auth Token jest niezbędny do komunikacji pomiędzy naszym czujnikiem a aplikacją Blynk.

Na kolejnym ekranie zobaczymy pusty ekran naszej nowo utworzonej aplikacji:

<img src="images/pusty_projekt.png" height=450px></img>

Po kliknięciu w dowolne miejsca na czarnym tle ukażą się możliwe do dodania komponenty z tzw. Widget Box-a.

<img src="images/dostepne_widgety.png" height=450px></img>

Przesuwając listę dostępnych komponentów w dół znajdziemy komponent opisany jako 'Labeled Value' - klikamy w niego.

<img src="images/opisany_wyswietlacz.png" height=450px></img>

W polu Title - wpisujemy 'Temperatura', następnie klikamy w pole 'PIN' i wybieramy Virtual oraz V1.

<img src="images/wybierz_pin.png" height=450px></img>

W polu opisanym jako Label wpisujemy '/pin/'. Po wykonaniu wszystkich tych kroków powinniśmy uzyskać coś takiego:

<img src="images/konfiguracja_pierwszego_komponentu.png" height=450px></img>

Następnie dodajemy drugi taki sam komponent (Labeled Value):

<img src="images/dodany_nastepny_komponent.png" height=450px></img>

Podobnie jak poprzednio nadajemy mu nazwę, tym razem 'Wilgotność' oraz wybieramy pin - Virtual oraz V2:

<img src="images/wybierz_pin2.png" height=450px></img>

Na koniec skonfigurowany komponent powinien wyglądać tak:

<img src="images/konfiguracja_drugiego_komponentu.png" height=450px></img>

Oba komponenty po pomyślnej konfiguracji będą wyglądać następująco:

<img src="images/dodane_dwa_komponenty.png" height=450px></img>

Klikamy w przycisk 'Play' oznaczający, że chcemy uruchomić naszą aplikację.

<img src="images/uruchom.png" height=450px></img>


### Podłączenie czujnika temperatury i wilgotności do modułu Node MCU ###

![DHT11](images/dht-11.jpg)

![Node MCU](images/esp8266-nodemcu.png)

|Node MCU  |     DHT11|
|----------|---------:|
|D2 (GPIO4)|  Data Out|
|3V3       |       VCC|
|G (GND)   |    Ground|

### Konfiguracja komputera do zaprogramowania modułu Node MCU ###

* Pobierz odpowiednią dla Twojego systemu wersję Arduino IDE z https://www.arduino.cc/en/software i zainstaluj
* Uruchom Arduino IDE i dodaj biblioteki dla Node MCU. W tym celu musimy podać ścieżkę gdzie znajdują się biblioteki do ESP8266. Otwórz menu Plik -> Preferencje, następnie w polu "Dodatkowe adresy URL do menedżera płytek" wpisz:

```
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

i kliknij OK.

![Dodatkowe Biblioteki](images/dodatkowe_biblioteki.png)

Następnie klikamy w menu Narzędzia -> Płytka: "Arduino Yun" -> Menedżer płytek...

![Menedzer Plytek](images/menedzer_plytek.png)

W okienku, które się pojawi wpisujemy ESP8266 i wybieramy esp8266 by ESP8266 Community i klikamy Instaluj.

![Biblioteka ESP8266](images/esp_8266_lib.png)

Kliknij ponownie w menu Narzędzia -> Płytka: "Arduino Yun" -> ESP8266 Boards -> i wybierz NodeMCU 1.0 (ESP-12E Module)

TODO rysunek

Sprawdz czy wszystkie pozostałe opcje są zgodne z tym co na poniższym rysunku.

TODO rysunek

* Podłącz poprzez kabel microUSB Node MCU do komputera (sterowniki powinny zainstalować się automatycznie, gdyby tak się jednak nie stało - pobieramy sterowniki do CH340 np. z tej strony http://www.arduined.eu/ch340-windows-8-driver-download i instalujemy je ręcznie)


### Program do odczytu temperatury i wilgotności  ###

* Przekopiuj ponizszy kod źródłowy programu do okna 'sketch' w Arduino IDE:

```
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

char auth[] = "Blynk_auth_token";
 
char ssid[] = "Nazwa_sieci_(SSID)";
char pass[] = "Haslo";

#define BLYNK_PRINT Serial
 
#define DHTPIN 4          // D2
#define DHTTYPE DHT11     // DHT 11
 
DHT dht(DHTPIN, DHTTYPE);
BlynkTimer sensorTimer;
 
void sendSensorData() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
 
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Zly odczyt z sensora DHT.");
    return;
  }
  
  Blynk.virtualWrite(V1, temperature);
  Blynk.virtualWrite(V2, humidity);
}
 
void setup() {
  Serial.begin(115200);
 
  Blynk.begin(auth, ssid, pass);
 
  dht.begin();
 
  sensorTimer.setInterval(1000L, sendSensorData);
}
 
void loop() {
  Blynk.run();
  sensorTimer.run();
}
```

* Następnie kliknij w przycisk Wgraj (zostaniesz dodatkowo poproszony o zapisanie projektu, wybierz dowolną lokalizację i kliknij Zapisz) - co spodowuje skompilowanie i wgranie programu do pamięciu modułu Node MCU.
![Wgraj Program](images/wgraj_program.png)



### Ewentualne problemy ###

* W razie problemów z wgraniem aplikacji na Node MCU, konieczne może być wgranie nowego firmware. W tym celu pobierz program ESP8266Flasher (ze strony: https://github.com/nodemcu/nodemcu-flasher/raw/master/Win64/Release/ESP8266Flasher.exe), uruchom program, wybierz port do którego podpięty jest Twój Node MCU i kliknij FLUSH. Program automatycznie pobierze najnowsze oprogramowanie dla Twojego modułu, następnie wgra je do pamięci modułu.
