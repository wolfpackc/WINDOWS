# Token Stealing con técnicas Potato — conclusiones

El objetivo final de una técnica Potato suele ser **conseguir ejecutar un proceso con la identidad `NT AUTHORITY\SYSTEM`**. La característica importante es que normalmente no partimos de `SeDebugPrivilege`, como en un robo directo de tokens, sino de un proceso cuyo token contiene **`SeImpersonatePrivilege`**.

## 1. Qué cambia entre las distintas Potato

La fase inicial depende de la variante Potato utilizada. JuicyPotato, PrintSpoofer, GodPotato y otras variantes emplean mecanismos diferentes para conseguir que un componente privilegiado de Windows se conecte o autentique contra un endpoint controlado por el proceso atacante.

Lo que cambia principalmente entre variantes es **cómo se consigue esa conexión privilegiada**.

Una vez que el cliente privilegiado se ha conectado, comienza la fase de impersonación.

## 2. Flujo conceptual

```text
Cliente privilegiado / SYSTEM
        ↓
conexión al endpoint controlado
        ↓
ImpersonateNamedPipeClient()
        ↓
el hilo obtiene un Impersonation Token
        ↓
OpenThreadToken()
        ↓
acceso al token de impersonación
        ↓
DuplicateTokenEx(..., TokenPrimary, ...)
        ↓
nuevo Primary Token
con identidad SYSTEM
        ↓
CreateProcessAsUser()
o CreateProcessWithTokenW()
        ↓
nuevo proceso
        ↓
NT AUTHORITY\SYSTEM
```

## 3. Qué ocurre durante la impersonación

Cuando se ejecuta `ImpersonateNamedPipeClient()`, el proceso **no pierde su token primario original**. Lo que ocurre es que el hilo que realiza la impersonación recibe un **Impersonation Token**, que pasa a ser su identidad efectiva para determinadas comprobaciones de seguridad.

Por tanto:

```text
Proceso
→ continúa teniendo su Primary Token original

Hilo impersonado
→ tiene además un Impersonation Token
→ actúa temporalmente con la identidad del cliente
```

La impersonación afecta al hilo que está impersonando; no reemplaza automáticamente el Primary Token del proceso completo.

## 4. De Impersonation Token a Primary Token

El problema es que un **Impersonation Token no se utiliza directamente como token primario de un proceso nuevo**. Por eso se abre primero mediante `OpenThreadToken()` y posteriormente se utiliza `DuplicateTokenEx()` para crear una copia de tipo `TokenPrimary`.

Ese nuevo Primary Token sí puede utilizarse explícitamente para crear otro proceso bajo la identidad impersonada mediante APIs como `CreateProcessAsUser()` o `CreateProcessWithTokenW()`.

> **Impersonar permite que un hilo actúe temporalmente como el cliente; duplicar el token como Primary Token permite materializar esa identidad en un proceso nuevo.**

## 5. Caso SYSTEM

En una Potato, si el cliente impersonado es SYSTEM, el resultado final buscado es:

```text
Impersonation Token SYSTEM
        ↓
Primary Token SYSTEM
        ↓
nuevo proceso
        ↓
NT AUTHORITY\SYSTEM
```

## 6. Conclusión final

Por tanto, la cadena final de muchas variantes Potato es conceptualmente muy parecida:

1. Conseguir que un contexto privilegiado se conecte al endpoint controlado.
2. Impersonar ese contexto privilegiado.
3. Obtener el Impersonation Token del hilo.
4. Duplicarlo como `TokenPrimary`.
5. Crear un nuevo proceso utilizando ese Primary Token.

Lo que suele diferenciar unas Potato de otras está principalmente en la fase anterior: **el mecanismo utilizado para conseguir que un contexto privilegiado se conecte y pueda ser impersonado**.

### Resumen mental

**Forzar una autenticación privilegiada → impersonar → obtener el token del hilo → duplicarlo como Primary Token → crear un nuevo proceso con esa identidad.**
