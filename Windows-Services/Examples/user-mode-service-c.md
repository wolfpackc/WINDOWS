# Ejemplo de servicio de Windows en C

Un **servicio Win32 de user mode** puede ser un `.exe` escrito en C que se registra con el Service Control Manager y responde a órdenes como iniciar o detenerse.

## Código mínimo

```c
#include <windows.h>

SERVICE_STATUS        ServiceStatus;
SERVICE_STATUS_HANDLE StatusHandle;

void WINAPI ServiceCtrlHandler(DWORD CtrlCode)
{
    if (CtrlCode == SERVICE_CONTROL_STOP)
    {
        ServiceStatus.dwCurrentState = SERVICE_STOPPED;
        SetServiceStatus(StatusHandle, &ServiceStatus);
    }
}

void WINAPI ServiceMain(DWORD argc, LPTSTR *argv)
{
    StatusHandle = RegisterServiceCtrlHandler(
        "MiServicio",
        ServiceCtrlHandler
    );

    if (!StatusHandle)
        return;

    ZeroMemory(&ServiceStatus, sizeof(ServiceStatus));

    ServiceStatus.dwServiceType = SERVICE_WIN32_OWN_PROCESS;
    ServiceStatus.dwCurrentState = SERVICE_RUNNING;
    ServiceStatus.dwControlsAccepted = SERVICE_ACCEPT_STOP;

    SetServiceStatus(StatusHandle, &ServiceStatus);

    // Aquí estaría el trabajo real del servicio.
    while (ServiceStatus.dwCurrentState == SERVICE_RUNNING)
    {
        Sleep(1000);
    }
}

int main(void)
{
    SERVICE_TABLE_ENTRY ServiceTable[] =
    {
        { "MiServicio", ServiceMain },
        { NULL, NULL }
    };

    StartServiceCtrlDispatcher(ServiceTable);

    return 0;
}
```

## Qué ocurre en este código

La parte más importante al comienzo del flujo es:

```c
StartServiceCtrlDispatcher(ServiceTable);
```

Con esta llamada, el `.exe` le indica básicamente a Windows: **“soy un servicio; conecta mi proceso con el Service Control Manager”**.

El SCM utiliza la tabla proporcionada para localizar la función principal del servicio y, cuando inicia el servicio, llama a:

```c
ServiceMain(...)
```

Dentro de `ServiceMain`, el servicio registra una función que recibirá órdenes del SCM:

```c
RegisterServiceCtrlHandler(
    "MiServicio",
    ServiceCtrlHandler
);
```

De esta forma, el proceso puede reaccionar a controles como `SERVICE_CONTROL_STOP`. El servicio también informa al SCM de su estado mediante `SetServiceStatus`.

El flujo conceptual es:

```text
MiServicio.exe
      │
      │ StartServiceCtrlDispatcher()
      ▼
     SCM
      │
      │ inicia el servicio
      ▼
 ServiceMain()
      │
      ├── registra ServiceCtrlHandler()
      ├── informa de su estado
      └── realiza su trabajo
```

Todo este código continúa ejecutándose en **user mode**. El hecho de utilizar `ServiceMain`, `RegisterServiceCtrlHandler` y el Service Control Manager no convierte el proceso en código de kernel. Sigue siendo un `.exe`; simplemente está estructurado para funcionar bajo el modelo de servicios de Windows.

## Idea clave

El lenguaje tampoco determina si algo es una aplicación, un servicio o un driver. Los tres pueden encontrarse escritos en C o C++. Lo que cambia es **el entorno de ejecución y las APIs utilizadas**:

```text
Aplicación .exe      → user mode
Servicio .exe        → user mode, gestionado por SCM
Driver .sys          → kernel mode
```
