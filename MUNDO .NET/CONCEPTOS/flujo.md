partimos de un codigo escrito en un lenguaje de programacion como C# que se encuentra dentro del mundo de .net

despues con un compilador unico de ese lenguaje obtenemos un codigo que solemos nombrar como lenguaje intermedio(es portable) que no sirve para nada en realidad

luego utlizamos otra caja de herramientas que se llama runtime ( que la solemos encontrar dentro del sdk de .net) este runtime entre las muchas cosas que hace es 
con una herramienta conocida como JIT se coge el lenguaje intermedio y lo compila haciendo que obtengamos un codigo valido.
Genera código nativo en memoria (RAM) que el proceso ejecuta.

Cuando el programa se cierra, ese código desaparece.
