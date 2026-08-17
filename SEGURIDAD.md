## Que se guarda donde y por que

### En el navegador

- `boveda_access`: cookie `httpOnly`, `secure` en produccion, `sameSite=lax` y con
	una vida de 15 minutos. Contiene un JWT firmado que se envia automaticamente
	en cada request y permite autorizar la operacion actual.
- `boveda_refresh`: cookie con las mismas protecciones, pero con una vida maxima
	de 8 horas. Sirve unicamente para obtener un nuevo access token. No debe estar
	disponible para JavaScript, por eso se usa `httpOnly`.


### En el servidor y SQLite

- `usuarios`: identidad, rol, sucursal, cedula y `hashContrasena`. La contrasena
	original nunca se guarda; el hash permite verificarla sin poder recuperarla.
- `sesiones`: `id`, usuario asociado, fechas de creacion y ultimo acceso,
	`revocadaEn` y `refreshActual`. Esta fila hace que la sesion sea revocable: un
	JWT que aun no expiro tampoco sirve si la sesion fue revocada.
- `refreshActual`: solo guarda el identificador (`refreshId` o `jti`) del refresh
	vigente, no el JWT completo. Se compara contra el refresh presentado y se
	reemplaza cada vez que hay una rotacion.
- `solicitudes`: datos de negocio, como sucursal, cuenta destino, monto,
	justificacion y estado. Se guarda en el servidor para aplicar autorizacion y
	doble control en cada operacion.
- `auditoria`: eventos append-only, actor, fecha y metadatos. Se usa para
	trazabilidad y para dejar evidencia de un intento de reuso de refresh.

El secreto `SESSION_SECRET` no se guarda en la base ni en el navegador: debe
existir como secreto de configuracion del servidor. Permite firmar y verificar
los JWT.

## Que pasa si se filtra el refresh

Un refresh filtrado debe considerarse una credencial de sesion. Quien lo posea
puede presentarlo al servidor mientras su firma y su expiracion sean validas.
El `access` de vida corta limita la ventana de uso directo, pero no elimina el
riesgo del refresh.

### Con rotacion y deteccion de reuso

1. El atacante puede usar el refresh robado una primera vez si todavia es el
	 vigente. El servidor emite un refresh nuevo y cambia `refreshActual`.
2. El refresh robado queda invalidado inmediatamente. Si el atacante vuelve a
	 presentarlo, su `refreshId` ya no coincide con `refreshActual`.
3. Ese reuso se trata como indicio de robo: se rechaza la solicitud, se revoca
	 la sesion completa y se registra `reuso_de_refresh` en la auditoria. Tambien
	 deja de funcionar el refresh nuevo que pudiera tener el usuario legitimo,
	 obligandolo a iniciar sesion otra vez.

La rotacion reduce el tiempo util del token robado y permite detectar que existe
una copia. Hay una condicion de carrera posible si el mismo refresh se usa casi
simultaneamente por el cliente legitimo y el atacante; por eso, en produccion,
la actualizacion del refresh vigente debe ser atomica.

### Sin rotacion

El mismo refresh seguiria siendo valido despues de cada uso. Un atacante podria
renovar repetidamente la sesion y obtener nuevos access tokens durante toda la
vida del refresh, hasta que expire la sesion o alguien la revoque manualmente.
El servidor no tendria forma de distinguir el uso legitimo del uso del atacante,
porque ambos presentarian exactamente el mismo token. No habria deteccion de
reuso ni un evento confiable para disparar la revocacion automatica.

En resumen: la rotacion no hace que una filtracion sea inocua, porque el primer
uso del token robado puede funcionar, pero convierte los usos posteriores en una
señal detectable y permite cortar la sesion. Sin rotacion, la filtracion mantiene
una credencial reutilizable hasta su expiracion o revocacion.