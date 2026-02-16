
# 📘 Cuestionarios de Seguridad Windows

> Archivo de autoevaluación personal sobre Windows Internals, Tokens, Procesos, Privilegios y Elevación.


---

## 🧠 Tema: Tokens de Acceso

### 🔹 Preguntas rápidas
* [ ] ¿que me dices de un proceso con un token de acceso y que le pasa a los hilos de ese proceso?

* [ ] ¿token primario es lo mismo que token de acceso?
* [ ] ¿puedo crear un token yo mismo?
* [ ] ¿Qué es un token de acceso?
* [ ] ¿Qué información contiene un token?
* [ ] ¿Qué diferencia hay entre token primario y token de suplantación?
* [ ] ¿Quién tiene token primario: proceso o hilo?
* [ ] ¿Puede un hilo tener token primario?

---

### 🔹 Preguntas de razonamiento

* [ ] ¿Por qué no puedo crear un token arbitrario con privilegios máximos?
* [ ] ¿Por qué necesito duplicar un token antes de usarlo?
* [ ] ¿Qué verifica el kernel al usar un token?

---

### 🔹 Preguntas de flujo

* [ ] Flujo para crear proceso con token elevado (método 2)
* [ ] Flujo para suplantar hilo (método 1)
* [ ] Diferencias prácticas entre método 1 y método 2

---

### 📝 Respuestas (ocultas)

<details>
<summary>Ver respuestas</summary>

**Token de acceso:**
Estructura que describe quién eres y qué puedes hacer.

**Token primario:**
Pertenece a procesos.

**Token de suplantación:**
Se asigna a hilos.

**Método 1:**
Duplicar token → SetThreadToken → CreateProcess

**Método 2:**
Duplicar token → CreateProcessWithToken

</details>

---

### 🎯 Mini-reto mental

> Si un CMD usuario puede crear un CMD SYSTEM usando un token duplicado, ¿por qué no es un fallo de diseño?

Respuesta mental:
Porque Windows valida que tengas privilegios para **usar** ese token.

---

### 📊 Autoevaluación

Marca lo que ya puedes explicar sin mirar:

* [ ] Puedo explicar qué es un token
* [ ] Puedo explicar primario vs suplantación
* [ ] Puedo explicar método 1
* [ ] Puedo explicar método 2
* [ ] Puedo explicar por qué no se crean tokens desde cero

---

---

## 🧠 Tema: Handles

### 🔹 Preguntas

* [ ] ¿Qué es un handle?
* [ ] ¿Quién crea los handles?
* [ ] ¿Un archivo tiene token?
* [ ] ¿Un archivo tiene DACL?
* [ ] ¿Qué se puede hacer con un handle de proceso?

---

<details>
<summary>Ver respuestas</summary>

Un handle es una referencia a un objeto del kernel.
Los tokens pertenecen a procesos, no a archivos.
Los archivos tienen descriptores de seguridad.

</details>

---

---

## 🧠 Tema: SRM (Security Reference Monitor)

### 🔹 Preguntas

* [ ] ¿Qué compara el SRM?
* [ ] ¿Qué pasa si el token no cumple la DACL?
* [ ] ¿Quién manda: token o proceso?

---

---

## 📈 Progreso general

* [ ] Tokens
* [ ] Handles
* [ ] Procesos e hilos
* [ ] Privilegios
* [ ] Elevación
* [ ] Persistencia
