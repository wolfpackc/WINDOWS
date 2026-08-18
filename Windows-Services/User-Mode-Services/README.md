# User Mode Services

Un **servicio de Windows en user mode** suele ser un programa `.exe`, igual que una aplicación normal. La diferencia no está en el lenguaje ni necesariamente en la extensión, sino en **cómo se integra con Windows, quién lo administra y para qué está pensado**.

Una aplicación normal, como `Fortnite.exe`, suele iniciarla directamente un usuario y normalmente está vinculada a su sesión y a una interacción concreta. Un servicio está pensado para realizar una función persistente o de sistema: puede arrancar automáticamente con Windows, continuar funcionando aunque ningún usuario abra una interfaz y ser detenido o reiniciado de forma controlada.

Ejemplos típicos de servicios son:

- antivirus;
- Windows Update;
- servidores web;
- servidores de bases de datos;
- impresión;
- Bluetooth;
- servicios de red;
- copias de seguridad.

La idea que conviene memorizar es:

> **Aplicación y servicio son dos formas distintas de organizar procesos de user mode: la aplicación está orientada normalmente a una sesión y a la interacción del usuario; el servicio está orientado a realizar una tarea persistente o de sistema y su ciclo de vida lo administra el Service Control Manager.**

## Relación con el kernel

Tanto una aplicación normal como un servicio siguen estando en **user mode**. Ambos interactúan continuamente con el kernel mediante APIs y syscalls. Por ejemplo, abrir archivos, reservar memoria, crear hilos o utilizar sockets termina provocando operaciones dentro del kernel.

Sin embargo, eso no significa que puedan acceder arbitrariamente a memoria de kernel o ejecutar código en Ring 0. Para determinadas operaciones relacionadas con dispositivos pueden comunicarse con drivers mediante interfaces como `DeviceIoControl` e IOCTLs, pero es el driver `.sys` que se ejecuta en kernel mode quien realiza la operación privilegiada.

```text
Proceso.exe / Servicio.exe
          │
          │ APIs / syscalls / I/O
          ▼
      Windows kernel
          │
          ▼
      Drivers .sys
```

Por tanto, **ser un servicio no concede acceso especial al kernel**. La diferencia esencial está en la administración del proceso, su ciclo de vida y la identidad de seguridad con la que el SCM puede iniciarlo.