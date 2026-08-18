# Token Stealing — Potato: conclusiones

## Idea principal

El objetivo habitual de estas técnicas es **ejecutar un proceso como `NT AUTHORITY\SYSTEM`**, uno de los contextos más privilegiados de Windows en modo usuario.

Hay dos caminos conceptuales distintos para llegar a ese resultado:

### 1. Token stealing directo

Parte normalmente de un proceso elevado que dispone de **`SeDebugPrivilege`**.

```text
Administrador elevado
        ↓
SeDebugPrivilege
        ↓
abrir un proceso SYSTEM
        ↓
abrir su token
        ↓
duplicarlo como token primario
        ↓
crear un proceso con ese token
        ↓
NT AUTHORITY\SYSTEM
```

La idea es sencilla: **vamos directamente a buscar un token SYSTEM que ya existe**. `SeDebugPrivilege` resulta especialmente importante para superar la barrera inicial de acceso al proceso privilegiado; después se solicitan sobre el token los derechos necesarios para consultarlo o duplicarlo.

### 2. Técnicas Potato



Potato parte normalmente de una situación diferente: el proceso **no dispone de `SeDebugPrivilege`**, pero su token contiene **`SeImpersonatePrivilege`**. Esto es común en determinados servicios y cuentas de servicio, aunque tambien en cmd como admin.

```text
Proceso/cuenta de servicio limitada
        ↓
SeImpersonatePrivilege
        ↓
conseguir una conexión de un contexto privilegiado
        ↓
impersonar al cliente
        ↓
obtener un Impersonation Token SYSTEM
        ↓
duplicarlo como Primary Token
        ↓
crear un proceso
        ↓
NT AUTHORITY\SYSTEM
```

La forma más fácil de recordarlo es:

> **Token stealing directo: voy a buscar el token de SYSTEM.**  
> **Potato: hago que un contexto privilegiado venga hacia mí y entonces lo impersono.**

## Qué hace realmente la impersonación

Cuando un hilo impersona a un cliente, **el proceso no pierde ni sustituye su Primary Token original**. El hilo obtiene además un **Impersonation Token**, que pasa a utilizarse como su identidad efectiva para determinadas comprobaciones de seguridad.

```text
Proceso → mantiene su Primary Token original
Hilo impersonado → usa temporalmente un Impersonation Token
```

Para crear un proceso nuevo con esa identidad, el token de impersonación puede duplicarse como **`TokenPrimary`** y utilizarse después en la creación del nuevo proceso.

La conclusión importante es:

> **Impersonar permite actuar temporalmente con otra identidad; duplicar ese token como Primary Token permite materializar esa identidad en un proceso nuevo.**

## Por qué existen tantas variantes Potato

JuicyPotato, RoguePotato, PrintSpoofer, GodPotato, JuicyPotatoNG, etc. se diferencian sobre todo en **el mecanismo empleado para provocar la conexión o autenticación privilegiada inicial**: named pipes, RPC, DCOM, Spooler u otros componentes.

La parte final es conceptualmente muy parecida:

```text
conexión privilegiada
        ↓
impersonación
        ↓
token SYSTEM
        ↓
duplicación como Primary Token
        ↓
proceso SYSTEM
```

## Cuentas de servicio

Una **cuenta de servicio** es una identidad utilizada por servicios o aplicaciones de Windows, no necesariamente por una persona interactiva. No todas son SYSTEM ni tienen el mismo nivel de privilegio.

Ejemplos típicos:

- `LOCAL SERVICE`: relativamente limitada.
- `NETWORK SERVICE`: relativamente limitada y capaz de autenticarse en red como el equipo en determinados contextos.
- cuentas específicas de IIS, SQL Server, agentes, backups, etc.
- `NT AUTHORITY\SYSTEM`: extremadamente privilegiada localmente.

El escenario interesante para Potato es precisamente una **cuenta de servicio limitada que posea `SeImpersonatePrivilege`**. Si ya estamos ejecutando como SYSTEM, la escalada local a SYSTEM deja de tener sentido porque el objetivo ya se ha alcanzado.

Además, Potato **no concede `SeImpersonatePrivilege`**. El ejecutable debe comenzar bajo un proceso cuyo token ya contenga ese privilegio. Si un servicio vulnerable ejecuta código o crea un proceso hijo, ese proceso puede heredar el contexto de seguridad del servicio y convertirse en el punto de partida de Potato.

## User mode frente a kernel mode

Las técnicas Potato son **user mode (Ring 3)**. Trabajan mediante tokens, handles, impersonación y APIs de Windows; **no modifican directamente `_EPROCESS.Token` ni otras estructuras internas del kernel**.

En kernel mode, en cambio, el enfoque estudiado consiste en acceder directamente a estructuras como `_EPROCESS` y modificar referencias internas al token. Es una metodología diferente aunque el objetivo final pueda ser parecido.

## Lo confirmado con GodPotato en el laboratorio

La práctica con GodPotato confirmó el modelo teórico: partiendo de **`NT AUTHORITY\NETWORK SERVICE` / Servicio de red**, la herramienta creó un endpoint controlado, utilizó RPC/DCOM para conseguir una conexión privilegiada, realizó impersonación, encontró un token SYSTEM utilizable y creó un nuevo proceso bajo `NT AUTHORITY\SYSTEM`.
<img width="845" height="601" alt="image" src="https://github.com/user-attachments/assets/e81d9521-49ac-4792-b378-1652de328823" />
El log observado seguía esencialmente este flujo:

```text
NETWORK SERVICE
        ↓
endpoint / named pipe
        ↓
RPCSS / DCOM
        ↓
conexión privilegiada
        ↓
impersonación
        ↓
token SYSTEM encontrado
        ↓
nuevo proceso como SYSTEM
```

También quedó claro que **conseguir ejecución como SYSTEM y obtener una consola plenamente interactiva son problemas distintos**. GodPotato puede elevar correctamente y crear un proceso SYSTEM aunque la entrada/salida de la consola (`stdin/stdout/stderr`) no quede conectada de forma usable.

## Conclusiones finales

- El objetivo típico es alcanzar **`NT AUTHORITY\SYSTEM`**.
- **`SeDebugPrivilege`** es la pieza clave del método directo para acceder a procesos privilegiados y posteriormente trabajar con sus tokens.
- **`SeImpersonatePrivilege`** es la pieza clave de las Potato basadas en impersonación.
- Un privilegio puede estar presente y deshabilitado; si está ausente del token, no puede añadirse simplemente con `AdjustTokenPrivileges`.
- Potato no suele ser el primer paso: normalmente se utiliza **después de conseguir ejecución en un proceso o servicio cuyo token ya contiene `SeImpersonatePrivilege`**.
- La impersonación afecta inicialmente al **hilo**, no sustituye automáticamente el Primary Token del proceso.
- Las variantes Potato cambian principalmente en **cómo provocan la conexión privilegiada**; el tramo final de impersonación, duplicación del token y creación del proceso es similar.
- **Potato es una técnica de user mode**, no una modificación directa de estructuras del kernel.
- En la práctica con GodPotato se confirmó la escalada **NETWORK SERVICE → SYSTEM**; la interactividad de la shell es independiente de que la elevación haya funcionado correctamente.

### Resumen mental

```text
Método directo:
SeDebugPrivilege → abrir proceso SYSTEM → abrir token → duplicar → proceso SYSTEM

Potato:
SeImpersonatePrivilege → provocar conexión privilegiada → impersonar → duplicar token → proceso SYSTEM
```
