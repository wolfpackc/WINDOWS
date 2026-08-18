# Potato — pasos resumidos

Este documento resume de forma conceptual el flujo típico de una técnica Potato.

## 1. Punto de partida

Partimos de un proceso o servicio que controlamos de alguna forma. Puede tratarse de:

- un servicio vulnerable donde podemos ejecutar o inyectar código;
- un programa creado por nosotros y ejecutado en un contexto con privilegios suficientes;
- cualquier proceso cuyo token disponga de **`SeImpersonatePrivilege`**.

La condición realmente importante es que el contexto desde el que ejecutamos la técnica tenga **`SeImpersonatePrivilege`**.

## 2. Crear un canal de comunicación controlado

Nuestro proceso crea un endpoint IPC controlado, normalmente conceptualizado como una **tubería (named pipe)** u otro mecanismo equivalente según la variante Potato.

```text
Nuestro proceso
    ↓
crea endpoint / pipe controlado
```

## 3. Conseguir que un contexto privilegiado se conecte

La técnica fuerza o provoca que un proceso o servicio privilegiado de Windows se comunique con ese endpoint.

El cliente que se conecta puede ejecutarse, por ejemplo, como:

```text
NT AUTHORITY\SYSTEM
```

u otra cuenta de servicio privilegiada.

```text
Proceso privilegiado
        ↓
se conecta al pipe / endpoint
        ↓
nuestro proceso recibe la conexión
```

## 4. Impersonar al cliente

En el momento en que el cliente privilegiado está conectado, nuestro hilo utiliza la capacidad de impersonación para adoptar temporalmente su identidad.

Conceptualmente:

```text
Cliente privilegiado conectado
        ↓
ImpersonateNamedPipeClient()
        ↓
nuestro hilo obtiene
un Impersonation Token
```

El proceso original conserva su Primary Token. La identidad privilegiada se aplica al **hilo que está impersonando**.

## 5. Obtener el token impersonado

Una vez que el hilo está actuando como el cliente privilegiado, se obtiene acceso a su token de impersonación.

```text
Hilo impersonando SYSTEM
        ↓
OpenThreadToken()
        ↓
Impersonation Token SYSTEM
```

## 6. Convertirlo en un Primary Token

Ese token de impersonación se duplica como un token primario.

```text
Impersonation Token SYSTEM
        ↓
DuplicateTokenEx(..., TokenPrimary, ...)
        ↓
Primary Token SYSTEM
```

## 7. Crear un proceso nuevo con ese token

Finalmente, el nuevo Primary Token se utiliza para iniciar otro proceso, por ejemplo una terminal.

```text
Primary Token SYSTEM
        ↓
CreateProcessAsUser()
o CreateProcessWithTokenW()
        ↓
cmd.exe
        ↓
nueva terminal como SYSTEM
```

## Resumen completo

```text
Proceso con SeImpersonatePrivilege
        ↓
crea pipe / endpoint controlado
        ↓
provoca que un servicio privilegiado se conecte
        ↓
cliente SYSTEM se conecta
        ↓
impersonamos su identidad en un hilo
        ↓
obtenemos el Impersonation Token
        ↓
lo duplicamos como Primary Token
        ↓
creamos un proceso con ese token
        ↓
cmd.exe / nuevo proceso
        ↓
NT AUTHORITY\SYSTEM
```

> **Idea clave:** la Potato no consiste simplemente en localizar un token de SYSTEM y copiarlo. El patrón consiste en conseguir que un contexto privilegiado se autentique contra un endpoint controlado, impersonar esa identidad y después convertirla en un Primary Token utilizable para crear un nuevo proceso.
