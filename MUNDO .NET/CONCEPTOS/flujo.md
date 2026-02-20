partimos de un codigo escrito en un lenguaje de programacion como C# que se encuentra dentro del mundo de .net

despues con un compilador unico de ese lenguaje obtenemos un codigo que solemos nombrar como lenguaje intermedio(es portable) que no sirve para nada en realidad

luego utlizamos otra caja de herramientas que se llama runtime ( que la solemos encontrar dentro del sdk de .net) este runtime entre las muchas cosas que hace es 
con una herramienta conocida como JIT se coge el lenguaje intermedio y lo compila haciendo que obtengamos un codigo valido.
Genera código nativo en memoria (RAM) que el proceso ejecuta.

Cuando el programa se cierra, ese código desaparece.



Un dato interesante es que aunque necesitemos compiladores distintos para cada lenguaje de prog, el runtime(clr) es capaz de coger e interpretar todos los los lenguajes intermedios que reciba
Sí, **vas perfecto hasta ahí**. Vamos a dejarlo clarísimo paso a paso.

---

# 🔹 1️⃣ Lenguajes en .NET

* Dentro del ecosistema .NET puedes usar:

  * **C#**
  * **F#**
  * **VB.NET**
  * Etc.

* Cada lenguaje tiene **su propio compilador**:

  * C# → `csc`
  * F# → `fsc`
  * VB → `vbc`

Por eso, si quieres programar en C#, necesitas un **SDK que incluya el compilador de C#**. Lo mismo para F#.

---

# 🔹 2️⃣ Compilación → IL

* El compilador **convierte tu código fuente en IL (Intermediate Language / CIL)**
* El IL se guarda en un `.exe` o `.dll`
* Este IL **es portable**: funciona en cualquier runtime compatible con .NET
* No depende de la CPU ni del sistema operativo (aunque luego se ejecuta usando un runtime específico)

---

# 🔹 3️⃣ El runtime lo entiende todo

* Aunque los compiladores de C#, F# o VB sean distintos, **todos generan IL compatible con el CLR**
* Por eso el runtime **puede ejecutar IL de cualquier lenguaje .NET**
* Esto es lo que permite que, por ejemplo, una librería escrita en F# pueda usarse desde C# sin problemas

---

# ✅ Resumen mental

```text
Lenguaje (.NET)
   ↓ Compilador específico
IL (.exe / .dll)
   ↓ Runtime (CLR)
Código nativo → CPU ejecuta
```
