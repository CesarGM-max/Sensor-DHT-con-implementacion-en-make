# Sensor DHT11 con implementación en Make

**Arduino UNO R4 WiFi + DHT11 + Make.com + Telegram**

- **Alumno:** Cesar Jesus Gamez Mendoza
- **Materia:** Desarrollo Sustentable
- **Profesor:** Mtro. Miguel Ángel Barrón Hernández
- **Institución:** Instituto Tecnológico de Mazatlán

## Descripción

Un Arduino UNO R4 WiFi lee la temperatura y la humedad de un cuarto con un sensor **DHT11**. Los valores se muestran en una página web ("Monitor del cuarto") que la misma placa sirve en la red local. Cuando la temperatura llega a **30 °C o más**, se enciende un LED y el Arduino envía la temperatura a un **webhook de Make.com**. El escenario de Make manda una alerta por **Telegram**:

> ⚠️ El cuarto está muy caliente (temperatura). Enciende el aire acondicionado.

## Objetivo

Monitorear la temperatura de un cuarto y avisar automáticamente por Telegram cuando está muy caliente, para encender el aire acondicionado solo cuando hace falta y usar la energía de forma más eficiente.

## Material utilizado

- Arduino UNO R4 WiFi
- Sensor de temperatura y humedad DHT11
- LED rojo y resistencias
- Protoboard y cables Dupont
- Laptop (Arduino IDE y Make.com) y teléfono con Telegram

## Conexiones

- **DHT11:** VCC → 5V, GND → GND, DATA → pin digital 2
- **LED:** pin digital 13 (con su resistencia) → GND

## Código

[`Codigo/Leer_DHT11.ino`](Codigo/Leer_DHT11.ino)

El programa hace lo siguiente:

1. Se conecta a la red WiFi (2.4 GHz) y levanta un servidor web en el puerto 80.
2. Cada 2 segundos lee la temperatura y la humedad del DHT11.
3. Si la temperatura es de 30 °C o más, enciende el LED del pin 13.
4. La **primera vez** que la temperatura cruza los 30 °C, envía por HTTPS el JSON `{"temperatura": valor}` al webhook de Make. No vuelve a avisar hasta que la temperatura baje y vuelva a subir, así se evitan mensajes repetidos.
5. Muestra la temperatura, la humedad y el estado (Normal / CALIENTE) en la página "Monitor del cuarto", que se actualiza cada 5 segundos.

> En el archivo, el nombre y la contraseña de la red WiFi y el ID del webhook se reemplazaron por textos de ejemplo. Hay que poner los propios antes de cargar el programa.

## Escenario en Make (blueprint)

[`Codigo/Leer DTH11.blueprint.json`](Codigo/Leer%20DTH11.blueprint.json)

El blueprint es la exportación del escenario **"Leer DTH11"** de Make.com (⋯ → *Export Blueprint*). Se puede volver a cargar en Make con *Import Blueprint*. Contiene dos módulos:

1. **Webhooks – Custom webhook** (`gateway:CustomWebHook`, módulo 1). Es el disparador del escenario. Recibe el JSON que envía el Arduino y define la estructura de datos con un solo campo, `temperatura` (número). `maxResults: 1` indica que procesa un mensaje por ejecución.
2. **Telegram Bot – Send a Text Message or a Reply** (`telegram:SendReplyMessage`, módulo 2). Usa la conexión del bot "Temperatura" y envía al chat configurado en `chatId` el texto:
   `⚠️ El cuarto está muy caliente ({{1.temperatura}}) Enciende el aire acondicionado.`
   La expresión `{{1.temperatura}}` toma el valor de temperatura que llegó al módulo 1. `parseMode` vacío significa que se envía como texto plano.

En la configuración general (`metadata`), `instant: true` indica que el escenario se ejecuta en cuanto llega un dato al webhook (*Immediately as data arrives*); `maxErrors: 3` desactiva el escenario después de 3 errores seguidos, y `zone: us2.make.com` es el servidor de Make de la cuenta.

> En el archivo se reemplazó el ID del chat de Telegram por `TU_CHAT_ID`. Hay que poner el propio al importarlo.

![Escenario en Make](Imagenes/Escenario%20Make%20Leer%20DTH11.png)

## Imágenes

![Circuito en protoboard](Imagenes/Circuito%20en%20protoboard.jpg)

![Arduino UNO R4 WiFi](Imagenes/Arduino%20UNO%20R4%20WiFi.jpg)

![Armado del circuito](Imagenes/Armado%20del%20circuito.jpg)

![Alertas en Telegram](Imagenes/Alertas%20en%20Telegram.jpg)

## Video

[Ver video en YouTube](https://youtu.be/Ljq2hAXILz0)

Archivo del video: [`Video/Funcionamiento Sensor DHT11.mp4`](Video/Funcionamiento%20Sensor%20DHT11.mp4)

## Resultados

[`Resultados/Resultados.pdf`](Resultados/Resultados.pdf)

- Con temperatura normal, el monitor web mostró **29.80 °C y 50 % de humedad**, con estado "Normal" y el LED apagado.
- Al calentar el sensor subió a **30.20 °C y 69 % de humedad**; el estado cambió a "CALIENTE (LED encendido)" y llegó la alerta a Telegram.
- El bot recibió alertas con temperaturas entre **30.0 y 34.2 °C**, siempre por encima del umbral de 30 °C.

![Monitor web en estado caliente](Resultados/Monitor%20web%20-%20estado%20caliente.png)

## Conclusiones

La práctica integró un sensor físico con un servicio en la nube. El Arduino UNO R4 WiFi detectó cuándo el cuarto superaba los 30 °C, encendió el LED y, a través de Make.com, envió una alerta a Telegram en pocos segundos. Este tipo de automatización apoya el desarrollo sustentable, porque permite encender el aire acondicionado solo cuando realmente se necesita y evitar el consumo innecesario de energía.
