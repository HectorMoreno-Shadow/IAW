# Práctica 1 · Servidores, Proxiesy Certificados

## Dominios ficticios elegidos

### Marca 1: "marca1-i.practica"

- **Justificación del nombre:** Utilizo la extensión ".practica" para reflejar de que es un entorno de pruebas y el nombre "marca1-i" para identificar de que es la web interna.

### Marca 2: "marca2-p.practica"

- **Justificación del nombre:** Utilizo la extensión ".practica" para reflejar de que es un entorno de pruebas y el nombre "marca2-p" para representar la web pública.

## Eleccion de MPM

**MPM seleccionado:** event

**Justificación:** He elegido event ya que crea varios procesos y de cada uno de ellos genera múltiples hilos donde cada hilo atiende a un cliente (cosa que prefork no tiene, con cual al tener muchas peticiones simultáneas, el rendiento cae). Es mucho más ligero que prefork ya que los hilos del mismo proceso comparten memoria y eso hace que consuma menos RAM y soporte más tráfico.

**¿Y por qué no worker?**

Bueno pues porque event tiene una ventaja más, y es que añade para cada proceso un hilo (aparte de los demás hilos) dedicado exclusivamente a escuchar eventos. Este hilo resuelve el problema del Keep-Alive que es cuando un cliente no envia datos, el hilo se queda bloqueado e inactivo esperando que el cliente envie datos, desaprovechando memoria.

Entonces este hilo nuevo que se añade al proceso resuelve esto, cuando el cliente no envia datos, el hilo se libera inmediatamente para atender a otros usuarios, dejando a esa conexión inactiva el hilo *listener*. Esto optimiza a la CPU y a la memoria RAM.
