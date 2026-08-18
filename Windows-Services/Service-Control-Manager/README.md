# Service Control Manager (SCM)

El **Service Control Manager (SCM)** es el componente de Windows encargado de registrar, iniciar, detener y administrar servicios. Un servicio puede configurarse para arrancar automáticamente, bajo demanda o según otras condiciones, y el SCM controla su ciclo de vida.

Una característica especialmente importante es que, al crear un servicio, puede indicarse **con qué cuenta debe ejecutarse**. Por ejemplo:

```text
MiServicio
├── LocalSystem
├── LocalService
├── NetworkService
└── DOMINIO\CuentaServicio
```

En un proceso normal, el token viene determinado por el contexto de seguridad disponible: el usuario autenticado, el token heredado del proceso padre, UAC y split token, impersonación o un token que ya se posea legítimamente. No se puede decidir simplemente que un `.exe` normal nazca como SYSTEM.

En un servicio, en cambio, la identidad de ejecución queda configurada de antemano y **el SCM crea el proceso con el token correspondiente a esa cuenta**. Si el servicio está configurado para ejecutarse como `LocalSystem`, el proceso nace directamente con un token de `NT AUTHORITY\SYSTEM`; no necesita robar, duplicar ni transformar un token previo para convertirse en SYSTEM.

La idea clave es:

> **Un proceso normal nace con un token derivado del contexto que ya tienes; un servicio puede tener definida de antemano una identidad de ejecución, y el SCM se encarga de crear el proceso con el token de esa identidad.**

## Relación con PsExec

PsExec utiliza esta infraestructura porque el SCM ofrece una forma estándar de registrar y arrancar un ejecutable como servicio. Conceptualmente:

```text
Administrador remoto
        │
        │ crea/configura servicio
        ▼
       SCM
        │
        │ cuenta: LocalSystem
        ▼
 servicio.exe
 Token = NT AUTHORITY\SYSTEM
        │
        │ crea proceso hijo
        ▼
     cmd.exe
 Token = NT AUTHORITY\SYSTEM
```

Si un servicio que ya se ejecuta como SYSTEM crea normalmente un proceso hijo, como `cmd.exe`, ese hijo se ejecutará bajo el mismo contexto de seguridad salvo que se utilicen explícitamente otras credenciales o un token diferente.

Por eso PsExec no necesita crear un servicio para entrar en kernel mode. Lo utiliza porque el **SCM permite registrar, arrancar y controlar un proceso bajo una identidad de servicio concreta**.