# Eliminamos contenedor previo (si existe)
docker rm -f mi-contenedor-ssh

# Desplegamos el contenedor en segundo plano
docker run -d -p 2222:22 --name mi-contenedor-ssh jaiderospina/retoctf:1.0

# Verificamos que esté corriendo
docker ps

---

### 🟦 **5. Conclusiones y Posibles Mejoras**

La combinación de **Docker**, **crunch** y **hydra** permite automatizar un ataque de fuerza bruta eficaz en entornos controlados, facilitando el aprendizaje sobre seguridad ofensiva y endurecimiento de servicios SSH.

---

#### 🛠️ **Mejoras sugeridas**

- ✅ **Implementar reglas de exclusión y throttling:** para reducir la carga en el servidor SSH y evitar bloqueos por exceso de intentos.
- ⏱️ **Añadir delays o limitar tareas paralelas en hydra** usando la opción `-t` para evadir mecanismos de detección o mitigación temprana.
- 🧠 **Probar ataques dirigidos** empleando reglas personalizadas de **Hashcat** y diccionarios más robustos que incluyan variantes léxicas realistas.

---

🔐 **Recomendación final:** Este tipo de ejercicios son ideales para fortalecer conocimientos sobre protección de servicios expuestos. Se recomienda siempre implementar autenticación basada en claves, políticas de bloqueo y monitoreo activo.
