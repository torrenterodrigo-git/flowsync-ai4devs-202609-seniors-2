## Purpose

Permitir que una persona cree una cuenta en FlowSync, inicie y cierre sesión con ella y consulte su perfil, de modo que solo quien tenga una sesión válida acceda a las zonas privadas de la aplicación.

## Requirements

### Requirement: Registro de cuenta por API

El sistema SHALL permitir crear una cuenta mediante `POST /api/v1/auth/signup` con `fullName`, `email`, `password` y `passwordConfirmation`, y SHALL devolver la cuenta creada junto con un token de acceso ya utilizable.

#### Scenario: Alta correcta

- **WHEN** se envía un email válido y no registrado, una contraseña de entre 8 y 32 caracteres, la misma contraseña en `passwordConfirmation` y un `fullName` de texto o `null`
- **THEN** la respuesta es un 200 con `{ data: { user, token } }`, donde `user` contiene `id`, `fullName`, `email`, `initials`, `createdAt` y `updatedAt` (nunca la contraseña) y `token` es una cadena opaca con la que ya pueden hacerse peticiones autenticadas

#### Scenario: Nombre ausente

- **WHEN** la petición no incluye la clave `fullName`
- **THEN** la respuesta es un 422 con un error de regla `required` sobre el campo `fullName`; para registrarse sin nombre hay que enviar `fullName: null`

#### Scenario: Email ya registrado

- **WHEN** se envía un email que ya existe exactamente igual en otra cuenta
- **THEN** la respuesta es un 422 con un error de regla `database.unique` sobre el campo `email`

#### Scenario: Mismo email con distintas mayúsculas

- **WHEN** existe una cuenta con `ana@x.com` y se registra `Ana@x.com`
- **THEN** el alta se acepta y se crea una segunda cuenta independiente, porque el email se compara distinguiendo mayúsculas y minúsculas y se guarda tal cual se envía

#### Scenario: Datos inválidos

- **WHEN** el email no tiene formato válido o supera 254 caracteres, la contraseña tiene menos de 8 o más de 32 caracteres, o `passwordConfirmation` no coincide con `password`
- **THEN** la respuesta es un 422 con `{ errors: [...] }`, un elemento por fallo, cada uno con `message`, `rule`, `field` y, si aplica, `meta` (por ejemplo `min` o `max`)

### Requirement: Inicio de sesión por API

El sistema SHALL emitir un token de acceso nuevo mediante `POST /api/v1/auth/login` cuando el email y la contraseña corresponden a una cuenta existente.

#### Scenario: Credenciales correctas

- **WHEN** se envían el email y la contraseña de una cuenta existente
- **THEN** la respuesta es un 200 con `{ data: { user, token } }`, con la misma forma que en el registro, y el token nuevo convive con los que la cuenta ya tuviera

#### Scenario: Credenciales incorrectas

- **WHEN** el email no corresponde a ninguna cuenta o la contraseña no es la suya
- **THEN** la respuesta es un 400 con un único error sin campo asociado, igual en ambos casos, de modo que no revela si el email existe

#### Scenario: Email con distintas mayúsculas

- **WHEN** la cuenta se registró como `ana@x.com` y se intenta entrar con `Ana@x.com`
- **THEN** la respuesta es un 400 de credenciales incorrectas

#### Scenario: Formato inválido

- **WHEN** falta el email o la contraseña, o el email no tiene formato válido
- **THEN** la respuesta es un 422 con los errores por campo; en el login no se valida la longitud de la contraseña

### Requirement: Acceso autenticado a la zona de cuenta

El sistema SHALL exigir un token de acceso válido en la cabecera `Authorization: Bearer <token>` para todas las rutas bajo `/api/v1/account`.

#### Scenario: Sin token o con token inválido

- **WHEN** se llama a `GET /api/v1/account/profile` o a `POST /api/v1/account/logout` sin cabecera `Authorization`, con un token inexistente o con uno ya revocado
- **THEN** la respuesta es un 401 con `{ errors: [...] }`

#### Scenario: Vigencia del token

- **WHEN** se usa un token emitido hace cualquier cantidad de tiempo que no se ha revocado
- **THEN** la petición se acepta, porque los tokens no caducan

#### Scenario: Respuestas siempre en JSON

- **WHEN** se hace cualquier petición a la API, sea cual sea su cabecera `Accept`
- **THEN** la respuesta, incluidos los errores, llega en JSON

### Requirement: Consulta del perfil por API

El sistema SHALL devolver los datos de la cuenta autenticada mediante `GET /api/v1/account/profile`.

#### Scenario: Perfil de la cuenta autenticada

- **WHEN** se llama con un token válido
- **THEN** la respuesta es un 200 con `{ data: user }`, con los mismos campos que en el registro y el login

#### Scenario: Iniciales con nombre de dos o más palabras

- **WHEN** el `fullName` de la cuenta es `ana maría lópez`
- **THEN** `initials` vale `AM`: la primera letra de las dos primeras palabras separadas por espacio, en mayúsculas

#### Scenario: Iniciales con nombre de una palabra

- **WHEN** el `fullName` de la cuenta es `Ada`
- **THEN** `initials` vale `AD`: las dos primeras letras en mayúsculas

#### Scenario: Iniciales sin nombre

- **WHEN** el `fullName` de la cuenta es `null` y su email es `rodrigo@gmail.com`
- **THEN** `initials` vale `RG`: la primera letra de lo que va antes de la arroba y la primera de lo que va después

### Requirement: Cierre de sesión por API

El sistema SHALL revocar el token usado en la petición cuando se llama a `POST /api/v1/account/logout`.

#### Scenario: Logout correcto

- **WHEN** se llama con un token válido
- **THEN** la respuesta es un 200 con `{ message: "Logged out successfully" }`, sin envoltorio `data`, y ese token deja de ser válido desde ese momento

#### Scenario: Otras sesiones de la misma cuenta

- **WHEN** la cuenta tiene varios tokens activos y se cierra sesión con uno de ellos
- **THEN** los demás tokens siguen siendo válidos

### Requirement: Navegación según el estado de sesión

La aplicación web SHALL mostrar las pantallas de acceso (`/login`, `/register`) solo a quien no tiene sesión y la pantalla de perfil (`/profile`) solo a quien la tiene.

#### Scenario: Visitante sin sesión en una zona privada

- **WHEN** alguien sin sesión abre `/profile` o cualquier dirección que no sea `/login` ni `/register`
- **THEN** acaba en la pantalla de inicio de sesión

#### Scenario: Persona con sesión en una pantalla de acceso

- **WHEN** alguien con sesión abre `/login` o `/register`
- **THEN** se le lleva a su perfil

#### Scenario: Dirección desconocida con sesión

- **WHEN** alguien con sesión abre una dirección que no existe
- **THEN** se le lleva a su perfil

### Requirement: Persistencia de la sesión en el navegador

La aplicación web SHALL recordar la sesión en el navegador entre recargas y cierres de pestaña, y SHALL comprobarla contra el servidor cada vez que la aplicación arranca.

#### Scenario: Recarga con sesión válida

- **WHEN** alguien con sesión recarga la página o vuelve a abrir la aplicación
- **THEN** ve un indicador de carga a pantalla completa mientras se comprueba la sesión y después sigue en la pantalla en la que estaba, sin volver a identificarse

#### Scenario: Sesión rechazada por el servidor

- **WHEN** la aplicación arranca con una sesión guardada que el servidor ya no reconoce
- **THEN** la sesión guardada se descarta y la persona llega al inicio de sesión con el aviso «Tu sesión ha caducado. Vuelve a iniciar sesión.»

#### Scenario: Servidor inaccesible al arrancar

- **WHEN** la aplicación arranca con una sesión guardada y el servidor no responde
- **THEN** la persona llega al inicio de sesión con el aviso «No se pudo conectar con el servidor. Comprueba que el backend está arrancado.», y al recargar con el servidor ya disponible recupera la sesión sin identificarse

#### Scenario: Error del servidor al arrancar

- **WHEN** la aplicación arranca con una sesión guardada y el servidor responde con un error interno
- **THEN** la persona llega al inicio de sesión con el aviso «Algo ha ido mal en el servidor. Inténtalo de nuevo en un momento.»

#### Scenario: Aviso de sesión perdida

- **WHEN** hay un aviso de sesión perdida y la persona pasa a la pantalla de registro
- **THEN** el aviso no aparece en el registro, pero vuelve a aparecer si regresa al inicio de sesión, y solo desaparece tras entrar o registrarse con éxito

### Requirement: Pantalla de inicio de sesión

La aplicación web SHALL ofrecer en `/login` un formulario con email y contraseña que abre la sesión y lleva al perfil.

#### Scenario: Contenido de la pantalla

- **WHEN** alguien sin sesión abre `/login`
- **THEN** ve la marca «FlowSync», el título «Inicia sesión», el texto «Entra con tu cuenta para volver a tus tareas.», los campos «Email» y «Contraseña», el botón «Entrar» y el enlace «¿Aún no tienes cuenta? Crea una», que lleva al registro

#### Scenario: Entrada correcta

- **WHEN** introduce credenciales válidas y pulsa «Entrar»
- **THEN** el botón pasa a «Entrando…» y queda deshabilitado mientras se espera, y al terminar la persona ve su perfil

#### Scenario: Credenciales incorrectas

- **WHEN** introduce un email o una contraseña que no corresponden a una cuenta
- **THEN** ve arriba del formulario el aviso «El email o la contraseña no son correctos.», los campos conservan lo escrito y el botón vuelve a estar disponible

#### Scenario: Campos vacíos o email mal formado

- **WHEN** envía el formulario con el email vacío o con un email sin formato válido, o con la contraseña vacía
- **THEN** la validación la hace el servidor, no el navegador, y bajo cada campo afectado aparece su error: «Falta rellenar el email.», «Introduce una dirección de email válida.» o «Falta rellenar la contraseña.»

#### Scenario: Servidor inaccesible

- **WHEN** envía el formulario y el servidor no responde
- **THEN** ve el aviso «No se pudo conectar con el servidor. Comprueba que el backend está arrancado.»

### Requirement: Pantalla de registro

La aplicación web SHALL ofrecer en `/register` un formulario para crear una cuenta que, si el alta sale bien, deja la sesión abierta y lleva al perfil.

#### Scenario: Contenido de la pantalla

- **WHEN** alguien sin sesión abre `/register`
- **THEN** ve la marca «FlowSync», el título «Crea tu cuenta», el texto «Regístrate para empezar a organizar el trabajo del equipo.», los campos «Nombre completo (opcional)», «Email», «Contraseña» con la ayuda «Entre 8 y 32 caracteres.» y «Repite la contraseña», el botón «Crear cuenta» y el enlace «¿Ya tienes cuenta? Inicia sesión»

#### Scenario: Alta correcta

- **WHEN** rellena datos válidos y pulsa «Crear cuenta»
- **THEN** el botón pasa a «Creando cuenta…» y queda deshabilitado mientras se espera, y al terminar la persona ve su perfil sin tener que iniciar sesión

#### Scenario: Nombre en blanco o con espacios

- **WHEN** deja el nombre vacío o solo con espacios, o lo escribe con espacios al principio o al final
- **THEN** la cuenta se crea sin nombre en el primer caso, y con el nombre sin esos espacios en el segundo

#### Scenario: Contraseñas distintas

- **WHEN** la contraseña y su repetición no coinciden
- **THEN** bajo «Repite la contraseña» aparece «Las contraseñas no coinciden.» sin que se llegue a contactar con el servidor

#### Scenario: Email ya registrado

- **WHEN** usa un email que ya tiene cuenta, escrito exactamente igual
- **THEN** bajo «Email» aparece «Ese email ya está registrado. Inicia sesión en su lugar.»

#### Scenario: Contraseña fuera de rango

- **WHEN** la contraseña tiene menos de 8 o más de 32 caracteres
- **THEN** la ayuda de longitud se sustituye bajo «Contraseña» por «la contraseña debe tener al menos 8 caracteres.» o «la contraseña no puede superar los 32 caracteres.», que empiezan en minúscula

#### Scenario: Email mal formado

- **WHEN** el email está vacío o no tiene formato válido
- **THEN** la validación la hace el servidor, no el navegador, y bajo «Email» aparece «Falta rellenar el email.» o «Introduce una dirección de email válida.»

### Requirement: Pantalla de perfil

La aplicación web SHALL mostrar en `/profile` los datos de la cuenta con sesión abierta y permitir cerrar la sesión desde ahí.

#### Scenario: Datos visibles

- **WHEN** alguien con sesión abre su perfil
- **THEN** ve un círculo con sus iniciales, su nombre completo (o «Sin nombre» si no lo tiene), su email, «Miembro desde» seguido de la fecha de alta en formato largo en castellano (por ejemplo «30 de septiembre de 2026») y el botón «Cerrar sesión»

#### Scenario: Cierre de sesión

- **WHEN** pulsa «Cerrar sesión»
- **THEN** la sesión se cierra en ese navegador, llega a la pantalla de inicio de sesión sin ningún aviso y la sesión deja de valer también en el servidor

#### Scenario: Cierre de sesión sin servidor

- **WHEN** pulsa «Cerrar sesión» y el servidor no responde
- **THEN** la sesión se cierra igualmente en el navegador y llega al inicio de sesión sin ver ningún error
