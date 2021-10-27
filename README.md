SENSOR DE TEMPERATURA

Los sensores de temperatura son componentes 
eléctricos y electrónicos que, en calidad de sensores, permiten medir
la temperatura mediante una señal eléctrica determinada. Dicha señal 
puede enviarse directamente o mediante el cambio de la resistencia. 
También se denominan sensores de calor o termosensores. Un sensor de temperatura 
se usa, entre otras aplicaciones, para el control de circuitos. 
Los sensores de temperatura también se llaman sensores de calor, detectores de calor
o sondas térmicas.
Recursos:
Modulo ESP32 – 30 Pines:

Creado y desarrollado por Espressif Systems, ESP32, una serie de microcontroladores de bajo costo y de bajo consumo con sistema en chip con Wi-Fi y Bluetooth de modo dual integrados. Este dispositivo tomaría los datos de nuestro sensor y así mismo enviando estos registros de temperatura a la nube.
Sensor de temperatura y humedad DTH11:

El DHT11 es un sensor digital de bajo costo de temperatura y humedad. El sensor de humedad utiliza un principio capacitivo mientras que el de temperatura utiliza un termistor. El sensor se integra fácilmente con un Arduino dado que contiene librerías desarrolladas específicamente para el mismo. Las lecturas del sensor sólo pueden realizarse cada dos segundos. El sensor Cuenta con 3 pines (Vcc, GND, OUT) conectados que se conectan a los pines de la tarjeta en el mismo orden a (3.3v, GND, Pin15).
Luz LED 5mm:

Diodos que disponen de un ánodo y un cátodo suelen utilizarse en dispositivos electrónicos como iluminaciones de semáforos, juguetes, publicidades e incluso como alarmas, la funcionalidad varía dependiendo del enfoque que le quiera brindar su usuario. El led cuenta con dos pines (ánodo y cátodo) que se conectan a los pines de la tarjeta en el mismo orden a (GND, Pin2 y Pin4).
IFTTT:

Como aplicación gratuita para programar funciones o tareas para realizarlas de forma automática utilizamos IFTTT una plataforma basada en servicio web dando como resultado la captura de la temperatura, humedad y a su vez enviando esta información a una BD como hoja de cálculo en drive y correos electrónicos de Google.

Es una plataforma en línea y aplicación Android. Permite automatizar procesos: desde inicios de sesión hasta programación con diferentes aplicaciones Android.
Thonny:

Thonny es un IDE de Python para principiantes, desarrollado en la Universidad de Tartu, Estonia, que tiene un enfoque diferente ya que su depurador está diseñado específicamente para el aprendizaje y la programación de enseñanza. ... Instalación sencilla; viene integrado con python 3.6
Lenguaje de programación Python Vers. 3.9

Python es un lenguaje de escritura rápido, escalable, robusta y de código abierto, ventajas que hacen de Python un aliado perfecto para la Inteligencia Artificial. Permite plasmar ideas complejas con unas pocas líneas de código, lo que no es posible con otros lenguajes.
El codigo:
El código ha sido escrito en el lenguaje de programación Python con un enfoque orientado a objetos, haciendo uso de una de las diferentes modalidades que nos brinda este lenguaje, en este caso se construyó en MicroPython y se hicieron uso de librerías enfocadas al funcionamiento del sensor DHT11 y sensor de Agua.
1.(Líneas 1 a 3) En primer lugar, se importan los módulos “machine, time, urequests, network, DHT11”, para usar los pines de la tarjeta, leer los diferentes sensores y establecer comunicación Wifi para envío de datos a internet a través del uso de diferentes Apis.
import network, time, urequests
from dht import DHT11
from machine import Pin, ADC
 
2.(Líneas 5 a 10) Creamos los diferentes objetos que me permitirán realizar la lectura de los diferentes sesnores; entre ellos el objeto “sensor_Hs” que utiliza el pin 36 de la tarjeta para medir la humedad del suelo; El “bojeto sensorDHT” que utiliza el pin 15 de la tarjeta y permitirá leer temperatura y humedad ambiente. Los Objetos “rojo y Verde” que utilizan los pines 2y 4 y que serán los indicadores de niveles altos o adecuados de las diferentes variables.
sensor_Hs = ADC(Pin(36))  # Creamis el objeto para realizar la lectura de humedad de suelo
sensor_Hs.width(ADC.WIDTH_12BIT)  # permite regular la precisión de lectura
sensor_Hs.atten(ADC.ATTN_11DB) # permite trabajar con 3.3v
sensorDHT = DHT11 (Pin(15))  # Creamos el objeto DHT11 y asignamos el pin apara leer temperatura
rojo = Pin(2, Pin.OUT)  # Creamos el objeto rojo y asignamos el pin
verde= Pin(4, Pin.OUT)  # Creamos el objeto verde y asignamos el pin
 
 
3.(Líneas 14 a 25) Seguido al paso anterior procedemos a crear una nueva función que va a permitir la conexión Wifi de la tarjeta ESP32 a la red especificada en el fragmento de código "conectaWifi".
def conectaWifi(red, password):
     global miRed
     miRed = network.WLAN(network.STA_IF)     
     if not miRed.isconnected():              #Si no está conectado…
          miRed.active(True)                   #activa la interface
          miRed.connect(red, password)         #Intenta conectar con la red
          print('Conectando a la red', red +"…")
          timeout = time.time ()
          while not miRed.isconnected():           #Mientras no se conecte..
              if (time.ticks_diff (time.time (), timeout) > 10):
                  return False
     return True
 
4.(Líneas 30 a 36) Luego procedemos a realizar el llamado a la función conectaWifi con las credenciales del usuario. Imprimimos los datos de conexión y creamos las variables url_1 y url_2 que nos va a permitir hacer uso de los applets creados en la página IFTTT:
url_1: Utiliza el applet que envía datos de los sensores a una hoja de cálculo de Google. url_2: Utiliza el applet que envía datos de los sensores a un correo electrónico de Google.
      if conectaWifi("red", "password"):
 
          print("Conexión exitosa!")
          print('Datos de la red (IP/netmask/gw/DNS):', miRed.ifconfig())
 
          url_1 = "https://maker.ifttt.com/trigger/sensor_dth/with/key/XXXXXXXXXXXXXXXX?"  #  Applet IFTTT
          url_2 = "https://maker.ifttt.com/trigger/correo_emergencia/with/key/XXXXXXXXXXXXXXXX?" # Applet IFTTT
5.(Líneas 40 a 51) Creamos la función While True; el ciclo infinito que inicia los sensores realiza la lactura de las variables y envia los datos a la url_1.
  while (True):
 
          time.sleep (4)
          sensorDHT.measure()
          temp=sensorDHT.temperature()
          hum=sensorDHT.humidity()
          hum_suelo =  int(sensor_Hs.read())
          print ("T={:02d} ºC, H={:02d}, Hs={:02d} = %".format (temp,hum, hum_suelo))
          respuesta_1 = urequests.get(url_1+"&value1="+str(temp)+"&value2="+str(hum)+ "&value3="+str(hum_suelo))      
          print(respuesta_1.text)
          print (respuesta_1.status_code)
          respuesta_1.close ()
6.(Líneas 53 a 83) Creamos las condicionales “if y else” que de acuerdo a los valores de las variables enviaran alertas a los correos electrónicos haciendo uso de la url_2 y encenderán o apagaran los leds indicadores.
  if hum_suelo < 300 :
 
              respuesta_2 = urequests.get(url_2+"&value1="+str(hum_suelo))      
              print(respuesta_2.text)
              print (respuesta_2.status_code)
              respuesta_2.close ()
              time.sleep(10)
              rojo.value(1)
              verde.value(0)
 
          elif temp> 25 :
 
              respuesta_2 = urequests.get(url_2+"&value1="+str(temp))      
              print(respuesta_2.text)
              print (respuesta_2.status_code)
              respuesta_2.close ()
              time.sleep(10)
              rojo.value(1)
              verde.value(0)
 
          else:
 
              rojo.value(0)
              verde.value(1)

https://youtu.be/xNT4KDUmxkk

