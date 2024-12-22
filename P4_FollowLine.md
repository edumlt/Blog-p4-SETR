# Blog-p4-SETR

## Introducción

En esta práctica debemos programar un Sigue-Líneas que se comunique a través de servicios MQTT con un servidor. El vehículo dispone de un Arduino Uno y una placa ESP32-Cam. Además, se dispone de tres sensores infrarrojos, con los que detectar la línea, y un sensor de ultrasonidos.

<div align="center">
<p style = 'text-align:center;'><img src="https://eu.elegoo.com/cdn/shop/files/elegoo-uno-r3-project-smart-robot-car-kit-v-40-with-camera-arduino-stem-kits-elegoo-shop-213405_58c1b112-f35e-4a45-b0e7-a5429f93200e_grande.jpg" alt="JuveYell" width="300px"> </p>
</div>

## Ka-chow.ino

Para lograr que el coche complete el circuito en el menor tiempo posible, hemos implementado un **Planificador RT** en la placa de Arduino. Hemos dividido el comportamiento en 2 tareas: el movimiento y la detección de obstáculo.

La tarea de movimiento es la que tiene la mayor prioridad. Esta, si debe moverse, obtiene el valor de los sensores infrarrojos en el siguiente orden: lado derecho, lado izquierdo y luego centro. La razón es que la línea del laboratorio es fina, por lo que si el coche está orientado correctamente en la dirección de la línea solo detecta el sensor central. Esto implica que si alguno de los sensores laterales está detectando, implica que el coche está desviado, por lo que debería reajustarse. Al mirar los lados primero, si detecta alguno, redireccionará al vehículo para orientarlo en la dirección correcta. Si ninguno de los sensores detecta la línea girará bruscamente en la dirección en la que se vio la línea por última vez.

La tarea de detección tiene una prioridad menor y una frecuencia de ejecución menor. El sistema obtiene la distancia a la que está el objetivo y si se encuentra a una distancia menor de la que se debe parar más un margen, para los motores para que se frene y indica que no debe seguir moviéndose.

A la hora del paso de mensajes hemos reducido lo que la placa Arduino enviará al ESP a un solo char, para que el envío necesite el menor gasto computacional posible. Si se debe enviar un número, tras el envío de este se envía el carácter **'}'**, como muestra de que se ha acabado el número.

## SerialESP.ino

La implementación del envío de mensajes, en la ESP32, consta de 3 bloques principales: El acceso a la red wifi y la conexión con el servidor MQTT, el ciclo de control de los mensajes a través del serial y el envío de mensajes en formato JSON.

Para la conexión con el servidor y el acceso a la red, se parametrizaron las direcciones, nombres de usuario y contraseñas en el fichero logging.h. Con esta información se configuró el cliente wifi y el publicador test para el servidor MQTT. El establecimiento de conexión se preparó en el bucle Setup con el fin de iniciar el programa únicamente si las conexiones se realizaban correctamente. Se configuró un mensaje **Ready** para dar la orden de inicio al Arduino.

Para el ciclo de control de mensajes, se hace uso del propio bucle de ejecución de la ESP; esto nos permite hacer las lecturas del Serial2 de manera más rápida. Añadiendo además un switch que agiliza la comparación de los char enviados por el Arduino, se consigue mayor eficiencia.

Para el envío de mensajes, se decidió usar la librería **_ArduinoJson_** para la construcción del mensaje y **_ThreadController_**, que se empleó para el envío del PING. Los mensajes que requerían de datos numéricos se realizaron en métodos separados, mientras que los mensajes únicamente compuestos de texto se organizaron en una función conjunta, evitando duplicidad de código.

## Video del funcionamiento

[Video del funcionamiento](https://urjc-my.sharepoint.com/:v:/g/personal/e_martint_2022_alumnos_urjc_es/Eem6IUV4nfZNp0JhhDiapRYBeuFc1vh4IGiIDqG7kt7Nvw)
