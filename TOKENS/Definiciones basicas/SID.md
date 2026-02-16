
# 🧩 ¿Qué es un SID?

SID = **Security Identifier**
Es un número único que Windows usa para identificar:

* Un usuario
* Un grupo
* Una cuenta del sistema

Ejemplo (simplificado):

```
S-1-5-21-1234567890-123456789-123456789-1001
```

Tú nunca trabajas con esto a mano normalmente. Windows sí.

---

# 👤 User SID (SID de usuario)

Es el **SID que representa a tu usuario concreto**.

Piensa:

👉 “Este soy yo”

Ejemplo mental:

```
Usuario: Eduardo
User SID: S-1-5-21-AAA-AAA-AAA-1001
```

Ese número identifica **solo a ti**.

Cuando un token dice:

```
User SID = S-1-5-21-AAA-AAA-AAA-1001
```

Windows sabe:

👉 Este token pertenece a Eduardo.

---

# 👥 Group SIDs (SIDs de grupo)

Son los grupos a los que perteneces.

Por ejemplo:

* Usuarios
* Administradores
* Escritorio remoto
* Invitados

Cada grupo tiene su propio SID.

Ejemplo:

```
Usuarios → S-1-5-32-545
Administradores → S-1-5-32-544
```

Tu token puede contener:

```
User SID: Eduardo
Group SIDs:
 - Usuarios
 - Administradores
 - Escritorio remoto
```

---

# 🔐 ¿Por qué son tan importantes los Group SIDs?

Porque casi nunca se dan permisos a usuarios individuales.

Se dan permisos a **grupos**.

Ejemplo:

Archivo:

```
Permitir lectura → Grupo Administradores
```

Windows no pregunta:

> ¿Eduardo puede leer?

Pregunta:

> ¿El token tiene el SID del grupo Administradores?

Si la respuesta es sí → acceso concedido.

---

# 🧠 Flujo mental sencillo

Cuando intentas abrir algo:

```
Token → contiene:
   User SID
   Group SIDs
```

Windows compara eso con:

```
ACL del objeto → lista de SIDs permitidos
```

Si alguno coincide → acceso permitido.

---

# 🎮 Ejemplo tipo videojuego

Personaje = Token
Nombre del personaje = User SID
Clanes a los que pertenece = Group SIDs

Puerta:

```
Solo Clan Magos puede pasar
```

Si tu personaje pertenece al clan Magos → pasas.

No importa tu nombre.

---

#  Diferencia en una frase

👉 **User SID = quién eres**
👉 **Group SIDs = a qué equipos perteneces**

---

# 🧠 Frase para memorizar

**Windows no mira nombres. Mira SIDs.**

---

Si quieres, el siguiente paso natural es explicarte **qué es una DACL y cómo usa esos SIDs**, que es donde todo encaja.
