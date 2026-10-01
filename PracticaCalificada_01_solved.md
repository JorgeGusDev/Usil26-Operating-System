```md
### PREGUNTA 1

**a) Procesos:** 4 procesos: Padre → Hijo 1 → Hijo 2 → Nieto.

**b) Orden de mensajes:**  
`Inicio → Hijo 1 → Hijo 2 → Nieto → Hijo 2 termina → Hijo 1 termina → Padre termina`

**c) Nieto antes de Hijo 1:** No. El Nieto se crea después de que Hijo 1 imprime su mensaje.

**d) `wait()`:**
- Padre → espera a Hijo 1.
- Hijo 1 → espera a Hijo 2.
- Hijo 2 → espera al Nieto.

**Cadena de finalización:**  
`Nieto → Hijo 2 → Hijo 1 → Padre`
```
