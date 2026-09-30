# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 5.5 High Plan Mode

**Herramienta:** Claude Code

```
Sos una analista de sistemas Senior y necesito revises el proyecto FlowSync en el vertical de cuentas y acceso. 
Explorá el código de ambas capas sin tocar nada:
- Backend: rutas, controladores, validadores, middlewares, modelo de usuario
- Frontend: pantallas de acceso, estado de sesión, protección de rutas

Quiero una analisis resumido. No escribas archivos, no propongas cambios.

```

**Qué salió:** Al utilizar el modo Plan dejo una archivo fuera del proyecto copn todo el analisis generado
Datos que salieron de ese analisis:
- El email no se normaliza (ni trim ni minúsculas)
- Tener el token en localStorage lo deja expuesto ante un ataque XSS.
- No hay un manejo global de los 401 durante la sesión: si revocan un token, el usuario recién se entera al recargar.



## Prompt 2

**Modelo:** Opus 5.5 High Plan Mode

**Herramienta:** Claude Code

```
Quiero que detectes inconsistencias y que las enumeres por modulo (Backend/Frontend)

```

**Qué salió:** Encontré 10 inconsistencias en el backend y 7 en el frontend. Las más serias son tres. Los tokens no caducan nunca y, en un caso concreto, quedan activos en el servidor sin que nadie pueda revocarlos. El email no se normaliza, así que puede haber cuentas duplicadas según las mayúsculas. Y el frontend traduce cualquier 400 como "credenciales incorrectas".


## Prompt 3

**Modelo:** Opus 5.5 High

**Herramienta:** Claude Code

```
Necesito que con el contexto que ya tienes generes la primera version de la especificacion en el archivo rft.md
Tanto para backend como para frontend revisa lo que pasa por la API y lo que se ve en pantalla, y nada que no sea cuentas y acceso.

Necesito que respetes el siguiente formato
Arriba, un ## Purpose de una o dos frases: para qué existe esta capability.
Debajo, ## Requirements, y colgando de él ### Requirement: en los que el sistema SHALL hacer algo.
Bajo cada requisito, al menos un #### Scenario: de cuatro almohadillas, con dos viñetas: **WHEN** y **THEN**. No hay casilla para el GIVEN: la precondición se mete dentro del WHEN.
En castellano, salvo las mayúsculas de la RFC.

Tenes que respetar las siguientes 3 reglas duras:
- Nada de ADDED, MODIFIED ni REMOVED. Eso es el vocabulario de un delta, y esto no es un delta: es la verdad actual del sistema. Si tu archivo tiene una de esas secciones, has escrito otra cosa.
- Solo comportamiento observable desde fuera. Ni un nombre de clase, ni un nombre de archivo, ni una ruta de código. En la API, observable es la petición y la respuesta. En la pantalla, observable es lo que una persona ve y puede hacer.
- No toques el código. Ni siquiera para arreglar lo que encuentres, y sobre todo para eso: lo que encuentres es material de la parte B.
```
**Qué salió:**
- API (5 requisitos): registro, inicio de sesión, acceso autenticado a la zona de cuenta, consulta del perfil y cierre de sesión.
- Pantalla (5 requisitos): navegación según el estado de sesión, persistencia de la sesión en el navegador, pantalla de inicio de sesión, pantalla de registro y pantalla de perfil.

Como pediste la verdad actual del sistema, las inconsistencias que encontré antes figuran como lo que el sistema hace hoy, sin juzgarlas:

el email distingue mayúsculas y permite cuentas duplicadas;
los tokens no caducan;
el cierre de sesión responde sin el envoltorio data;
las iniciales de una cuenta sin nombre salen del email ("RG");
algunos mensajes de error empiezan en minúscula.

## Parte B

- Cantidad de requisitos: 10
- Controlados con codigo: 0
- Inconsistencias:

      - el email distingue mayúsculas y permite cuentas duplicadas;
      - los tokens no caducan;
      - el cierre de sesión responde sin el envoltorio data;
      - las iniciales de una cuenta sin nombre salen del email ("RG");
      - algunos mensajes de error empiezan en minúscula.

- Bug: 

      - el email distingue mayúsculas y permite cuentas duplicadas;