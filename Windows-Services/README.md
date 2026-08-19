# Windows Services

Un **proceso normal** y un **servicio** no son dos categorías completamente distintas para el kernel: ambos siguen siendo procesos de **user mode**. La diferencia principal está en quién los inicia, cómo se administran y para qué existen. Un proceso normal, como `Fortnite.exe`, suele iniciarlo un usuario y está ligado a una aplicación concreta y a su sesión. Un servicio, en cambio, suele ser también un `.exe`, pero está registrado y gestionado por el **Service Control Manager (SCM)**, que puede iniciarlo automáticamente, detenerlo, reiniciarlo o mantenerlo funcionando aunque no haya ningún usuario interactuando con él. Esa es la razón de existir de servicios como antivirus, Windows Update, impresión, Bluetooth, servidores web o bases de datos.

Ambos siguen estando en **user mode**, por lo que pueden interactuar con el kernel mediante APIs, syscalls y mecanismos como `DeviceIoControl`, pero **no pueden acceder directamente y de forma arbitraria a la memoria del kernel ni ejecutar código en Ring 0**. Cuando necesitan que se realice una operación privilegiada, es el propio kernel o un driver `.sys` quien la ejecuta. Por tanto, un servicio no tiene acceso al kernel por el simple hecho de ser servicio; su principal diferencia respecto a una aplicación normal está en su integración con el SCM, en su ciclo de vida y en la identidad de seguridad bajo la que puede ejecutarse.

Una característica especialmente importante de los servicios es que, al crearlos, puede configurarse **con qué cuenta deben ejecutarse**. Por ejemplo, un servicio puede estar configurado para ejecutarse como `LocalSystem`, `LocalService`, `NetworkService` o una cuenta de usuario o de dominio concreta. En un proceso normal, el token suele venir determinado por el contexto que ya existe: el usuario autenticado, el token heredado del proceso padre, UAC y split token, impersonación o un token que ya se haya obtenido legítimamente. No se puede decidir simplemente que un `.exe` normal “nazca como SYSTEM”. En un servicio, en cambio, el SCM conoce previamente la identidad configurada y se encarga de crear el proceso con el token correspondiente. La idea clave es: **un proceso normal nace con un token derivado del contexto disponible; un servicio puede tener definida de antemano una identidad de ejecución y el SCM crea el proceso con el token de esa identidad**.

Eso explica el funcionamiento conceptual de **PsExec**. PsExec no crea un servicio para entrar en kernel mode, sino porque el SCM proporciona una forma estándar de registrar, arrancar y controlar un programa de forma remota. En el caso típico, el ejecutable que se instala en la máquina remota se registra como servicio y se configura para ejecutarse como **LocalSystem**. El SCM arranca entonces ese `servicio.exe` directamente con un token de `NT AUTHORITY\SYSTEM`; no es que el servicio robe o transforme su token en SYSTEM. Si ese proceso crea después un `cmd.exe` normalmente, el hijo se ejecutará bajo ese mismo contexto de seguridad, salvo que se indique explícitamente otro token o credenciales. Por eso PsExec puede terminar proporcionando una consola como SYSTEM: **el administrador tiene permiso para pedir al SCM que cree el servicio, el SCM lo inicia con la cuenta configurada y los procesos hijos pueden heredar ese contexto de seguridad**.

Cuentas de servicio
Una cuenta de servicio es una identidad utilizada por Windows para ejecutar servicios y procesos en segundo plano. No está pensada necesariamente para que una persona inicie sesión de forma interactiva.

Ejemplos habituales:

NT AUTHORITY\LOCAL SERVICE: relativamente restringida.
NT AUTHORITY\NETWORK SERVICE: limitada localmente, aunque puede autenticarse en red usando la identidad del equipo en determinados contextos.
NT AUTHORITY\SYSTEM / LocalSystem: extremadamente privilegiada localmente.
cuentas específicas creadas para IIS, SQL Server, agentes, backups u otros servicios.
Cuenta de servicio ≠ SYSTEM necesariamente.

Una cuenta de servicio puede ser limitada o extremadamente privilegiada dependiendo de su identidad y del token que tenga asociado.

## Estructura

- [`User-Mode-Services/`](./User-Mode-Services/) — qué es un servicio de user mode y por qué existe.
- [`Service-Control-Manager/`](./Service-Control-Manager/) — papel del SCM, cuentas de servicio y tokens.
- [`Examples/`](./Examples/) — ejemplo mínimo de un servicio Win32 escrito en C.
