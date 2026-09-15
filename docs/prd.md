# Product Requirements Document (PRD)

---

## Resumen ejecutivo

Kasajuana es una progressive web app para un estudio de cerámica que sustituye la gestión manual
de plazas y cobros (WhatsApp, Excel, transferencias sin conciliar) por un flujo autoservicio: las
alumnas reservan su plaza, gestionan su turno y pagan mensualidad, trimestralidad o bonos sueltos
desde el móvil; la dueña del estudio gestiona turnos y aforo desde un panel, sin perseguir pagos ni
generar facturas a mano.

El problema no es solo "reservar un hueco": es que hoy el control de quién ha pagado, quién tiene
turno fijo y quién puede ocupar un hueco libre vive repartido entre mensajes y memoria. La app lo
convierte en un sistema con estado: cada plaza tiene una alumna y un pago asociados, siempre
consultables.

## Problema que resuelve

- La dueña del estudio no tiene forma de saber, sin preguntar, qué huecos están libres un día
  concreto ni quién ha pagado ya la mensualidad del mes.
- Las alumnas con turno fijo no tienen manera de avisar que no van y liberar su plaza para que
  otra compañera la ocupe — el hueco se pierde.
- Las alumnas que solo quieren ir sueltas (bono) no tienen visibilidad de qué turnos tienen plaza
  libre.
- El cobro por transferencia exige conciliación manual mensual y no genera factura: cada cobro que
  hay que justificar ante la gestoría se reconstruye a mano.

## Usuario objetivo

**Alumna** — persona que asiste regularmente (mensualidad/trimestralidad con turno fijo semanal) o
de forma puntual (bono de sesiones sueltas) al estudio. Usa la app desde el móvil para ver su
turno, liberar su hueco si no puede ir, reservar huecos libres, comprar bonos y pagar.

**Administradora (dueña del estudio)** — gestiona los turnos disponibles y su aforo, consulta qué
alumnas tienen plaza en cada turno y su estado de pago, y usa las facturas generadas
automáticamente para su gestoría. Es la única administradora en esta versión (un solo estudio, sin
multi-tenant ni roles de profesorado adicionales).

*(No hay persona detallada con datos demográficos porque no hay investigación de usuario previa
todavía — si se necesita para decisiones de diseño, se define en una sesión aparte con datos
reales, no inventados.)*

---

## Funcionalidades core (MoSCoW)

### MUST

- **[M-01] Registro y login de alumna** — Dado un visitante sin cuenta, cuando se registra con
  email y contraseña, entonces recibe confirmación y accede a su área con turno vacío hasta que la
  administradora se lo asigne o compre un bono.
  *Negativo:* dado un email ya registrado, cuando lo envía, entonces ve un error inline y no se
  crea ninguna cuenta duplicada.

- **[M-02] Ver turno fijo asignado** — Dado una alumna con mensualidad/trimestralidad activa,
  cuando abre la app, entonces ve su turno semanal (día, hora, aforo) y el calendario de próximas
  sesiones.

- **[M-03] Liberar el hueco propio de un día** — Dado una alumna con turno fijo, cuando marca "no
  voy" para una fecha concreta, entonces su plaza de ese día queda visible como libre para otras
  alumnas y ella deja de contar como asistente ese día.
  *Negativo:* dado que intenta liberar un hueco fuera del plazo mínimo configurado (p. ej. mismo
  día), entonces ve un aviso y no se libera automáticamente (queda pendiente de que la
  administradora lo apruebe, si aplica).

- **[M-04] Reservar un hueco libre de otro turno** — Dado una alumna (con turno fijo que quiere
  cambiar de día, o con bono), cuando ve un hueco libre en otro turno, entonces puede reservarlo y
  queda como asistente confirmada de esa sesión.
  *Negativo:* dado que dos alumnas intentan reservar el mismo hueco a la vez, entonces solo una lo
  consigue y la otra ve el hueco como ya ocupado sin duplicar el aforo.

- **[M-05] Comprar bono de sesiones sueltas** — Dado una alumna sin mensualidad, cuando compra un
  bono (X sesiones) vía Stripe Checkout, entonces el pago se confirma, el bono queda activo con su
  saldo de sesiones, y recibe la factura correspondiente.

- **[M-06] Contratar y renovar mensualidad/trimestralidad** — Dado una alumna, cuando contrata una
  mensualidad o trimestralidad vía Stripe Billing, entonces se activa la suscripción recurrente,
  se le asigna el turno fijo elegido (si hay plaza) y el cobro se repite automáticamente en cada
  ciclo sin intervención manual.
  *Negativo:* dado que el turno elegido no tiene plaza libre, entonces no puede completar la
  contratación y ve qué turnos sí tienen aforo disponible.

- **[M-07] Factura automática por cada cobro** — Dado cualquier cobro (mensualidad, trimestralidad
  o bono), cuando Stripe lo procesa, entonces se genera una factura en PDF con los datos fiscales
  configurados, descargable por la alumna y consultable por la administradora.

- **[M-08] Panel admin — gestión de turnos y aforo** — Dado la administradora, cuando crea, edita o
  cierra un turno (día, hora, aforo máximo), entonces el cambio se refleja inmediatamente en lo que
  ven las alumnas al reservar.

- **[M-09] Panel admin — listado de alumnas, turnos y pagos** — Dado la administradora, cuando abre
  el panel, entonces ve cada alumna con su turno asignado, tipo de plan (mensualidad,
  trimestralidad, bono y saldo restante) y estado del último cobro.

- **[M-10] Notificación de confirmación** — Dado una reserva, liberación de hueco o cobro, cuando
  ocurre, entonces la alumna recibe un email de confirmación con el detalle.

### SHOULD

- **[S-01] Recordatorio antes de cada sesión** — Dado una alumna con sesión confirmada al día
  siguiente, cuando llega la hora configurada de aviso, entonces recibe un recordatorio por email.

- **[S-02] Historial de reservas y pagos** — Dado una alumna, cuando entra a su historial, entonces
  ve sus sesiones pasadas, pagos realizados y facturas descargables.

- **[S-03] Pausa temporal de mensualidad** — Dado una alumna con mensualidad activa, cuando
  solicita una pausa (p. ej. vacaciones) con antelación mínima configurada, entonces el cobro del
  periodo pausado no se ejecuta y su turno queda reservado a la vuelta.
  *Negativo:* dado que solicita la pausa fuera del plazo mínimo, entonces ve el motivo y no se
  aplica.

- **[S-04] Lista de espera en turno completo** — Dado un turno sin plazas libres, cuando una
  alumna se apunta a la lista de espera, entonces es notificada automáticamente si se libera un
  hueco, por orden de solicitud.

### COULD

- **[C-01] Notificaciones push de huecos de última hora** — Dado un hueco liberado el mismo día,
  cuando ocurre, entonces las alumnas con la PWA instalada reciben una notificación push (no solo
  email).

- **[C-02] Valoración de talleres** — Dado una alumna que ha asistido a una sesión, cuando quiere
  dejar una valoración breve, entonces puede hacerlo y queda visible solo para la administradora.

- **[C-03] Estadísticas de ocupación** — Dado la administradora, cuando abre un panel de
  estadísticas, entonces ve ocupación media por turno, tasa de huecos liberados y aprovechados, e
  ingresos por tipo de plan.

### WON'T (esta versión)

- Soporte multi-tenant (varios estudios o profesoras gestionando sus propios talleres).
- Métodos de pago distintos a Stripe (transferencia manual, Bizum, efectivo registrado en la app).
- App nativa iOS/Android — solo PWA instalable desde navegador.
- Gestión de nóminas, horarios de profesorado o RRHH del estudio.

---

## Flujos de usuario principales

**Alta y suscripción:** la alumna se registra, elige mensualidad o trimestralidad, selecciona un
turno con plaza libre, paga vía Stripe Billing y queda con turno fijo asignado y factura emitida.

**Gestión semanal del turno fijo:** cada semana la alumna ve su turno. Si no puede ir un día
concreto, lo marca como "no voy" y su plaza queda libre para otra alumna; por defecto sigue
teniendo su turno adjudicado la semana siguiente.

**Reserva de hueco suelto:** una alumna (con bono o queriendo cambiar de día puntualmente) consulta
qué turnos tienen huecos libres esa semana y reserva uno; si paga con bono, se descuenta una sesión
de su saldo.

**Compra de bono:** una alumna sin mensualidad compra un pack de sesiones vía Stripe Checkout,
recibe el bono activo con su saldo y la factura correspondiente, y puede reservar huecos sueltos
según disponibilidad.

**Gestión de turnos (administradora):** la administradora crea o ajusta turnos (día, hora, aforo)
desde el panel, y consulta en todo momento qué alumnas ocupan cada plaza y su estado de pago.

---

## Requisitos no funcionales

- **PWA instalable:** manifest + service worker, funciona como app desde el móvil (icono, pantalla
  completa). El listado de turnos propios debe poder consultarse con conectividad intermitente;
  reservar y pagar requieren conexión.
- **Seguridad de pagos:** ningún dato de tarjeta pasa ni se almacena en la app — todo el flujo de
  pago ocurre en Stripe Checkout/Billing (PCI DSS delegado a Stripe).
- **Protección de datos:** datos personales de alumnas (nombre, email, historial de asistencia y
  pago) tratados conforme a RGPD — acceso restringido a la propia alumna y a la administradora.
- **Concurrencia en reservas:** la reserva de un hueco libre debe ser atómica para evitar que dos
  alumnas ocupen la misma plaza (ver caso negativo de M-04).
- **Accesibilidad:** formularios de reserva y pago usables con teclado y lector de pantalla
  (WCAG AA como referencia, sin auditoría formal en el MVP).

---

## Fuera de alcance (explícito)

- Gestión de varios estudios o franquicias (ver WON'T).
- Migración automática de pagos históricos por transferencia: el histórico previo a Stripe se
  queda donde esté hoy (Excel/mensajes), no se importa a la app.
- Facturación con validez fiscal completa más allá de lo que ofrece Stripe Invoicing (si la
  gestoría necesita un formato distinto, se evalúa aparte, no bloquea el MVP).
