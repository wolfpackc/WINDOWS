
##  Qué hace ShellExecute con los tipos de archivo

Cuando llamas a:

```c
ShellExecuteEx(...)
```

(o cuando usas `start` o doble clic)

el Shell:

1️⃣ Mira el archivo o la acción solicitada

2️⃣ Comprueba su extensión (.txt, .pdf, .exe, etc.)

3️⃣ Consulta el Registro de Windows

4️⃣ Determina qué aplicación está asociada

5️⃣ Lanza esa aplicación con CreateProcess si corresponde


---

## ✅ Asociación de extensiones

En Windows las asociaciones se guardan en el Registro:

Ruta conceptual:

```
HKEY_CLASSES_ROOT
```

Allí se define:

* `.txt` → tipo de archivo
* `txtfile` → clase
* acción por defecto → open
* aplicación asociada → notepad.exe (por defecto)

Ejemplo mental:

```
.txt -> txtfile -> open -> notepad.exe
```

Por eso:

👉 Doble clic en TXT abre Notepad
👉 No necesitas escribir notepad.exe

---

## ✅ URLs

Las URLs también tienen asociación:

* `http://` → navegador
* `https://` → navegador

ShellExecute detecta el esquema:

```
https://google.com
```

 abre navegador por defecto

Porque en el Registro hay asociación para el protocolo.

---

##  Archivos sin asociación

Si no hay asociación:

* Windows no sabe qué programa usar
* Puede mostrar diálogo de “Abrir con…”
* O fallar

---

## 🧠 Diferencia con CreateProcess

CreateProcess:

* No entiende asociaciones
* Solo ejecuta ejecutables

##  Otro ejemplo

URL:

```
https://google.com
```

Flujo:

```
ShellExecute
   ↓
protocolo https
   ↓
asociación -> navegador
   ↓
CreateProcess(navegador)
```

Resultado: se abre Chrome/Edge/etc.
