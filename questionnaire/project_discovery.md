# Cuestionario de Descubrimiento del Proyecto
> La IA usa este fichero cuando no encuentra suficiente contexto para inicializar el Harness.
> Se hacen las preguntas de forma progresiva — primero las bloqueantes, luego las de detalle.

---

## NIVEL 1 — Preguntas bloqueantes (sin estas no se puede continuar)

Si no se puede responder alguna de estas desde los ficheros existentes, preguntar AL USUARIO:

```
1. ¿Cuál es el nombre del proyecto?

2. ¿Cuál es el propósito principal en una frase?
   (ej: "Una app web para gestionar inventario", "Un smart contract de NFTs interactivos")

3. ¿Qué tipo de proyecto es? (elige uno o varios)
   [ ] Web app (frontend + backend)
   [ ] API / Backend solo
   [ ] Smart contract / Blockchain
   [ ] Mobile app
   [ ] CLI / Tool
   [ ] Library / Package
   [ ] Data pipeline
   [ ] Otro: ___

4. ¿Cuál es el stack tecnológico principal?
   (lenguajes, frameworks, bases de datos, servicios cloud)

5. ¿Cuál es el estado actual?
   [ ] Solo una idea — no hay código
   [ ] Hay documentación pero no código
   [ ] Hay código en desarrollo (X% completado)
   [ ] Está en producción parcialmente
   [ ] Está completamente en producción
```

---

## NIVEL 2 — Preguntas de arquitectura (para definir agentes)

```
6. ¿Cuáles son las áreas de trabajo diferenciadas?
   (ej: frontend, backend API, base de datos, autenticación, pagos, deploy, testing)
   Lista las que apliquen:
   - Área 1: ___
   - Área 2: ___
   - Área 3: ___
   (añadir las que necesites)

7. ¿Cuáles son las dependencias entre áreas?
   (ej: "el frontend depende de que la API esté lista",
        "los tests dependen de que el contrato compile")

8. ¿Hay roles o personas diferentes trabajando en cada área?
   (esto ayuda a definir los límites de cada agente)
```

---

## NIVEL 3 — Preguntas de estado y urgencia

```
9. ¿Cuál es lo más urgente o bloqueante AHORA MISMO?
   (ej: "tomar la decisión de qué base de datos usar",
        "escribir los tests", "hacer el primer deploy en staging")

10. ¿Hay decisiones importantes pendientes que afectan al diseño?
    Lista las que conozcas:
    - Decisión 1: ___
    - Decisión 2: ___

11. ¿Hay bugs conocidos o deuda técnica documentada?
    [ ] No
    [ ] Sí — describir brevemente: ___

12. ¿Hay una fecha límite o milestone próximo?
    [ ] No
    [ ] Sí — fecha y descripción: ___
```

---

## NIVEL 4 — Preguntas de calidad y proceso

```
13. ¿Hay tests? ¿De qué tipo?
    [ ] No hay tests
    [ ] Tests unitarios
    [ ] Tests de integración
    [ ] Tests end-to-end
    [ ] Otro: ___

14. ¿Cómo se hace el deploy actualmente?
    [ ] Manual
    [ ] CI/CD (GitHub Actions, Jenkins, etc.)
    [ ] Docker / contenedores
    [ ] Sin deploy todavía
    [ ] Otro: ___

15. ¿Hay algún estándar de código o convención establecida?
    (ej: ESLint, Prettier, naming conventions, estructura de commits)
```

---

## NIVEL 5 — Preguntas para el CLAUDE.md

```
16. ¿Cuáles son los comandos más importantes del proyecto?
    - Build: ___
    - Test: ___
    - Dev server: ___
    - Deploy: ___

17. ¿Hay algo que la IA NO debe hacer nunca en este proyecto?
    (ej: "nunca modificar los archivos de producción directamente",
         "nunca commitear credenciales", "nunca hacer push a main sin PR")

18. ¿Hay algo específico que la IA debe saber sobre el dominio del negocio?
    (ej: "es un proyecto médico — HIPAA compliance", "es fintech — regulado")
```

---

---

## NIVEL 6 — Preguntas para la capa de contexto (.harness/context/)

> Estas preguntas son opcionales. Si el usuario no las responde, generar los ficheros con placeholders.
> Con las respuestas, los 7 ficheros de context/ se generan con contenido real desde el día 1.

```
19. ¿Qué problema concreto resuelve este proyecto para el usuario final?
    (ej: "Los equipos pierden tiempo buscando quién aprobó qué decisión técnica")
    → Genera: context/productContext.md y context/projectbrief.md

20. ¿Quiénes son los usuarios de este proyecto?
    (ej: "Desarrolladores de equipos de 5-20 personas usando IA para programar")
    → Genera: context/projectbrief.md (campo "Usuarios objetivo")

21. ¿Qué NO hará este proyecto (fuera de alcance)?
    (ej: "No es un gestor de proyectos, no reemplaza Jira, no tiene UI gráfica")
    → Genera: context/projectbrief.md (campo "Fuera de alcance")

22. ¿Hay restricciones no negociables?
    (ej: RGPD, licencias open-source, compatibilidad con versión X, zero-dependency)
    → Genera: context/projectbrief.md y context/techContext.md

23. ¿Cuáles son los módulos o áreas principales del proyecto?
    (ej: "frontend, backend, autenticación, pagos, tests, deploy")
    → Genera las filas de context/progress.md (una por módulo, todos ⬜ al inicio)

24. ¿Hay variables de entorno o secrets que el proyecto necesita?
    (SOLO el nombre de la variable, NUNCA el valor real)
    (ej: DATABASE_URL, API_KEY_STRIPE, JWT_SECRET)
    → Genera: context/techContext.md (tabla "Variables de entorno requeridas")
```

---

## Instrucción para la IA al usar este cuestionario

> Solo pregunta lo que NO puedas inferir de los ficheros existentes.
> Agrupa preguntas relacionadas para no hacer la conversación tediosa.
> Con los niveles 1 y 2 tienes suficiente para empezar — los niveles 3-5 se pueden completar después.
> El nivel 6 es para poblar la capa de contexto — si el usuario no responde, genera placeholders.
> Nunca preguntes más de 3-4 cosas a la vez.
