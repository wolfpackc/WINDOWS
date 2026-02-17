# ni de coña esta al mismo nivel que el token de acceso
Lo puede tener un hilo, no el proceso.

Es temporal.

Sirve para que ese hilo actúe como otro usuario.

-----

Por defecto, los hilos no tienen token propio.

Usan el token primario del proceso.

Si asignas un token de suplantación a un hilo → ese hilo ignora el token del proceso y usa el suyo.
