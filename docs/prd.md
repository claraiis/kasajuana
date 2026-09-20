# Product Requirements Document (PRD)

---

## Resumen ejecutivo

Kasa Juana es un estudio de cerámica en A Coruña, gestionado por Ángela. Esta PWA sustituye la
gestión manual del curso (WhatsApp, Excel, transferencias sin conciliar) por un sistema con
estado: cada alumna ve su turno, sus créditos de recuperación y sus pagos; Ángela gestiona aforo,
aprueba excepciones y factura sin reconstruir nada a mano.

El curso va de septiembre a junio, con grupos de hasta 7 alumnas por turno. Alrededor del curso
regular hay tres negocios más pequeños que la app también cubre: **talleres privados** (grupos
puntuales de hasta 6 personas), **regalos** (vales de experiencia individual) y la **reserva del
curso siguiente** (renovación de alumnas actuales + entrada de nuevas interesadas).

Lanzamiento objetivo: **1 de octubre de 2026**, con Ángela migrando alumnas, turnos, pagos,
recuperaciones y regalos ya existentes. Esta versión prioriza correos y avisos dentro de la app;
notificaciones push y renovación automática de cuotas quedan explícitamente para una fase
posterior.

## Problema que resuelve

- No hay visibilidad de quién ha pagado, qué huecos están libres un día concreto, ni cómo se
  controla el consumo y la caducidad de las recuperaciones.
- Los talleres privados y los regalos se gestionan por mensajes sueltos, sin plazos de aprobación
  ni seguimiento de qué falta pagar.
- Cada cobro que hay que justificar ante la gestoría se reconstruye a mano; no hay factura por
  defecto.

## Usuario objetivo

- **Administradora (Ángela)** — única administradora del estudio (un solo tenant). Da de alta
  alumnas, gestiona turnos y calendario, aprueba solicitudes y excepciones (talleres, cambios de
  turno, reembolsos, ajustes), registra pagos externos y aprueba facturas antes de emitirlas.
- **Alumna** — asiste a un turno fijo semanal (mensualidad o trimestralidad) durante el curso
  septiembre-junio. Gestiona su propia asistencia, usa sus recuperaciones, reserva talleres o
  regalos, paga y descarga sus facturas.
- **Interesada** — persona sin plaza, registrada gratis con sus preferencias de horario, a la
  espera de una vacante.
- **Cliente de taller o regalo** — persona, alumna o no, que contrata un taller privado o compra
  un regalo. Eso no le da plaza en el curso regular.

*(Sin persona con datos demográficos inventados: los roles arriba son los reales, tal como se
usan en el negocio.)*

---

## Funcionalidades core (MoSCoW)

### MUST

**Curso, alta y perfil**

- **[M-01] Alta de alumna por la administradora** — Dado que Ángela da de alta a una alumna con su
  email, turno(s) y fecha de incorporación, cuando la guarda, entonces la alumna recibe un correo
  para crear su contraseña y completar su perfil.

- **[M-02] Perfil con datos de facturación y alimentarios** — Dado una alumna completando su
  perfil (datos personales, de facturación, foto, alergias/intolerancias/preferencias
  alimentarias), cuando lo guarda, entonces esos datos de facturación y alimentarios son visibles
  solo para ella y para Ángela.

- **[M-03] Gestión de clases habilitada tras el primer pago** — Dado una alumna con perfil
  completo, cuando se registra su primer pago (por ella o por Ángela si es externo), entonces se
  habilita la gestión de clases; antes de pagar solo ve su turno y periodo asignado.
  *Negativo:* si Ángela ya registró un pago externo de ese periodo, no se le vuelve a pedir.

- **[M-04] Visibilidad social básica** — Dado cualquier alumna consultando un turno, cuando lo
  abre, entonces ve foto y nombre de las asistentes de ese turno, incluso de otros grupos.

- **[M-05] Cambio de turno fijo solo por la administradora** — Dado una alumna que quiere cambiar
  de turno fijo, cuando lo solicita, entonces solo Ángela puede aplicarlo: no hay autoservicio para
  esto.

- **[M-06] Catálogo de turnos y modalidades** — Dado el horario del curso (lunes 18:30; martes y
  miércoles 11:00 / 16:15 / 18:30; jueves 18:30) con aforo máximo de 7 alumnas por grupo, y las
  modalidades de 2h (un turno fijo, 70€/mes o 178€/trimestre) o 4h (dos turnos fijos, 125€/mes, sin
  trimestralidad), cuando se muestra disponibilidad, entonces refleja aforo real, no asientos
  físicos.

- **[M-07] Festivos sin recuperación por defecto** — Dado el calendario de festivos de A Coruña,
  cuando una sesión cae en festivo, entonces no genera recuperación, salvo que Ángela decida
  ofrecer una compensación expresa para ese festivo.

**Turno fijo, Ceramiga y recuperaciones**

- **[M-08] Confirmación por defecto y liberación** — Dado una alumna con turno habitual, cuando
  llega su sesión, entonces aparece confirmada por defecto; si marca "No puedo ir", su plaza queda
  liberada para esa sesión.

- **[M-09] Recordatorio semanal** — Dado cada domingo, cuando llega el aviso programado, entonces
  las alumnas reciben un recordatorio para revisar su asistencia de la semana.

- **[M-10] Plaza Ceramiga** — Dado un grupo incompleto con una plaza sin titular ("Ceramiga"),
  cuando se abren las reservas de la semana siguiente (domingo 20:00), entonces solo pueden
  reservarla alumnas con recuperaciones disponibles.

- **[M-11] Reserva de hueco liberado por una titular** — Dado el hueco que libera una titular,
  cuando otra alumna quiere reservarlo, entonces solo puede hacerlo desde las 08:00 del mismo día
  de la clase, aunque se haya liberado antes.

- **[M-12] Prioridad de la titular y plaza extra** — Dado que una titular vuelve a confirmar antes
  del inicio de su sesión y otra alumna ya había reservado ese hueco, cuando Ángela decide,
  entonces admite a ambas con una plaza extra excepcional (sin aumentar el aforo permanente) o
  aplica la prioridad de la titular y devuelve, sin penalización, la recuperación a la alumna
  desplazada.

- **[M-13] Recuperación automática por ausencia** — Dado que Ángela registra la asistencia real
  tras cada clase, cuando una alumna falta a su turno habitual (con o sin aviso, incluso tras
  confirmar), entonces se genera una recuperación sin exigir justificar el motivo.
  *Negativo:* no se puede generar por adelantado una recuperación de una ausencia futura, salvo que
  sea Ángela quien cancele la sesión.

- **[M-14] Requisitos y caducidad de la recuperación** — Dado una recuperación disponible, cuando
  la alumna quiere usarla, entonces necesita el mes de asistencia pagado (mensualidad o trimestre)
  y estar de alta activa — un mes en mantenimiento no permite recuperar; caduca el último día del
  segundo mes siguiente al de la clase perdida, y nunca más allá del 30 de junio.

- **[M-15] Cancelar una recuperación no la pierde** — Dado una recuperación reservada en otra
  sesión, cuando la alumna la cancela o falta a ella, entonces la clase vuelve a su saldo sin
  duplicarse y conservando su caducidad original.

- **[M-16] Faltas Ceramiga y bloqueo** — Dado que reservar Ceramiga y no acudir sin cancelar antes
  cuenta como falta (cancelar antes del inicio no cuenta, ni las ausencias al turno habitual o a
  huecos de otras titulares), cuando una alumna acumula dos faltas Ceramiga en un mes natural,
  entonces queda bloqueada para reservar Ceramiga el mes siguiente, se cancelan sus reservas
  Ceramiga de ese mes y se le devuelven los créditos con su caducidad original.
  *Negativo:* el bloqueo no afecta a su turno habitual ni a recuperaciones en huecos de otras
  alumnas.

**Calendario, alternativas y avisos**

- **[M-17] Calendario unificado** — Dado que Ángela gestiona clases regulares, sesiones
  alternativas, franjas de talleres privados, talleres reservados y eventos privados en un único
  calendario, cuando hay un bloqueo privado, entonces su contenido no se muestra a alumnas ni a
  clientes.

- **[M-18] Cancelación de clase genera crédito** — Dado que Ángela cancela una clase, cuando se
  procesa, entonces las titulares reciben un crédito; quien ya iba a recuperar en esa sesión
  recupera el mismo crédito sin duplicarlo, con un mes más de caducidad (cada cancelación adicional
  suma otro mes, sin superar el fin de curso).

- **[M-19] Sesión alternativa con prioridad** — Dado una sesión alternativa ofrecida tras una
  cancelación, cuando una alumna de la clase cancelada la consulta, entonces la ve con prioridad
  pero sin confirmación automática; confirmar exige tener un crédito disponible, y asistir consume
  una recuperación, no una clase extra.
  *Negativo:* la prioridad no aparta el crédito; otras alumnas pueden reservar los huecos no
  confirmados desde las 08:00 del día de la sesión, sujeto a esa prioridad.

- **[M-20] Estados visuales de la alternativa** — Dado el listado de una sesión alternativa,
  cuando se muestra, entonces distingue "por confirmar" (atenuado), "confirmada" (opacidad
  completa) y "plaza liberada" (hueco vacío), con etiqueta legible además de la diferencia visual.

- **[M-21] Avisos de hueco para una fecha concreta** — Dado una alumna apuntada a avisos de una
  sesión concreta (no una suscripción recurrente ni una reserva automática), cuando se libera una
  plaza de esa sesión, entonces recibe un correo indicando si ya puede reservarla (Ceramiga) o
  desde qué hora podrá hacerlo (hueco de titular, desde las 08:00 de ese día); al conseguir reserva
  en esa sesión, deja de recibir avisos para ella.
  *Negativo:* no se envía un segundo correo solo porque llegue la hora de apertura del hueco.

**Pagos y mantenimiento**

- **[M-22] Conceptos y métodos de cobro** — Dados los cinco conceptos de cobro (mensualidad,
  trimestralidad, mantenimiento de plaza, reserva del próximo curso, taller), cuando una alumna
  paga, entonces puede hacerlo por Stripe, transferencia o efectivo; los pagos por transferencia o
  efectivo los registra Ángela manualmente.

- **[M-23] Sin renovación automática** — Dado un periodo de mensualidad o trimestralidad, cuando
  termina, entonces no se renueva ni cobra automáticamente: la alumna elige y confirma cada pago.
  La app no permite comprar meses ya cubiertos ni un trimestre que sobrepase junio (trimestres
  fijos: sep-nov, dic-feb, mar-may; junio es siempre un mes suelto).

- **[M-24] Plazo de pago mensual e impago** — Dado que la cuota mensual se paga del día 1 al 7,
  cuando llega el día 8 sin pago, entonces no se confirman automáticamente las semanas siguientes y
  aparece "Pendiente de pago"; el turno sigue asignado y no se ofrece automáticamente a otras.
  Ángela decide si permite asistir sin pagar y cuándo retira la plaza.
  *Negativo:* las recuperaciones exigen el mes pagado, sin excepción por esta discrecionalidad.

- **[M-25] Alta a mitad de mes** — Dado un alta a mitad de mes, cuando Ángela acuerda un importe
  prorrateado (normalmente por transferencia), entonces registra mes cubierto, importe, fecha y
  método, y ese mes queda pagado sin exigir de nuevo el precio completo.

- **[M-26] Mantenimiento mensual de plaza** — Dado que una alumna solicita mantenimiento antes de
  que empiece el mes afectado, cuando lo confirma, entonces paga el 50% de su tarifa mensual (35€
  para 2h, 62,50€ para 4h), hasta un máximo de dos meses por curso; durante ese mes no asiste, no
  reserva recuperaciones ni genera créditos, y sus créditos previos conservan su caducidad sin
  ampliación.
  *Negativo:* desde el día 1 del mes afectado ya no se puede convertir en mantenimiento.

- **[M-27] Baja definitiva sin reembolso por defecto** — Dado una baja definitiva con trimestre ya
  pagado, cuando se procesa, entonces no genera reembolso ni saldo económico, salvo que Ángela
  autorice una excepción.

- **[M-28] Aprobación previa de todo movimiento de dinero** — Dado cualquier reembolso o
  generación de saldo económico que la app proponga (con importe y motivo), cuando se plantea,
  entonces no se ejecuta sin la aprobación previa de Ángela; editar o cancelar una reserva no mueve
  dinero por sí solo. En transferencia/efectivo se distingue aprobar una devolución de registrar
  que ya se hizo.

- **[M-29] Borrador de factura con aprobación** — Dado cualquier cobro registrado, cuando ocurre,
  entonces la app genera un borrador de factura que Ángela revisa y aprueba antes de emitirla y
  enviarla.

**Reserva del próximo curso e interesadas**

- **[M-30] Prioridad de renovación** — Dado el periodo del 15 al 30 de junio (ambos incluidos),
  cuando se abre, entonces se ofrece primero por correo la reserva a las alumnas actuales para su
  grupo asignado; los cambios de turno se acuerdan con Ángela.

- **[M-31] Señal de reserva** — Dado el proceso de renovación, cuando una alumna reserva su plaza,
  entonces paga una señal de 10€ (también en la modalidad de 4h) que se descuenta una sola vez del
  primer pago del nuevo curso, aunque Ángela registre ese pago por transferencia o efectivo.
  *Negativo:* si no paga los 10€ antes del cierre, el turno queda disponible; si reserva y no se
  incorpora, pierde la señal.

- **[M-32] Resumen de cierre para Ángela** — Dado el cierre del periodo de renovación, cuando
  termina, entonces Ángela recibe un resumen de grupos cubiertos y plazas disponibles.

- **[M-33] Registro de interesadas** — Dado una persona sin plaza, cuando se registra como
  interesada (gratis, con contacto y preferencias de horario), entonces queda distinguida de las
  alumnas que esperan recuperar una sesión concreta.

- **[M-34] Oferta de vacantes a interesadas** — Dado que quedan vacantes tras la renovación, cuando
  se libera un hueco, entonces las interesadas compatibles reciben un correo para reservar, por
  orden de reserva completada con pago; el hueco queda bloqueado brevemente durante el pago para
  evitar duplicidades.
  *Negativo:* si ya no queda plaza en el turno solicitado, se muestran otros turnos disponibles; si
  no hay ninguno, la interesada mantiene su interés registrado.

- **[M-35] Alta de interesada tras pago** — Dado una interesada que paga su reserva, cuando se
  confirma, entonces pasa a ser alumna con plaza reservada para el próximo curso; en septiembre
  paga la mensualidad normal, sin una nueva señal de 10€.

**Talleres privados**

- **[M-36] Oferta y franjas** — Dado un taller privado (1 a 6 participantes, 60€/participante, 3
  horas), cuando Ángela abre franjas en su calendario, entonces puede ofrecer varias el mismo día;
  domingos y fechas excepcionales se acuerdan personalmente, no se publican automáticamente.

- **[M-37] Solicitud y bloqueo provisional** — Dado una solicitud ordinaria (mínimo una semana de
  antelación, salvo excepción de Ángela) con fecha, grupo previsto y teléfono de la responsable,
  cuando se envía, entonces la franja queda bloqueada provisionalmente mientras Ángela la revisa.

- **[M-38] Aprobación con plazo de 48h** — Dado una solicitud pendiente, cuando pasan 48 horas sin
  respuesta de Ángela (con recordatorio a las 24h), entonces la solicitud caduca y se libera la
  franja.

- **[M-39] Pago que confirma el grupo** — Dado una solicitud aprobada, cuando la persona no paga
  los 60€ (una participante) en 24 horas, entonces se libera la franja; si paga, confirma la
  reserva del grupo completo sin suplemento por señal.

- **[M-40] Cambio de fecha requiere nueva aprobación** — Dado un taller ya reservado, cuando el
  cliente solicita cambiar de fecha, entonces requiere de nuevo la aprobación de Ángela; elegir una
  fecha disponible no confirma el cambio por sí sola.

- **[M-41] Cierre del grupo 24h antes** — Dado un taller a 24 horas de su inicio, cuando se
  comprueba su estado, entonces deben estar confirmados nombres, datos alimentarios y todos los
  pagos (la responsable puede pagar las plazas juntas o compartir enlaces individuales); si faltan
  pagos o datos, se avisa a Ángela, que decide manualmente — no se cancela automáticamente.

- **[M-42] Panel de estado del taller** — Dado un taller con participantes, cuando se consulta,
  entonces muestra asistentes, información pendiente, pagos recibidos y resto por pagar.

**Regalos**

- **[M-43] Compra de regalo** — Dado un regalo (60€, experiencia para una persona, sin fecha ni
  acompañantes elegidos), cuando se compra, entonces se entrega un código o enlace de uso.

- **[M-44] Uso del regalo** — Dado un código de regalo, cuando quien lo recibe solicita fecha (y
  opcionalmente acompañantes), entonces Ángela aprueba la solicitud; la plaza regalada aparece
  cubierta y las adicionales se pagan juntas o por acompañante, con datos y pagos completos 24h
  antes.

- **[M-45] Caducidad y condiciones del regalo** — Dado un regalo comprado, cuando se consulta,
  entonces es transferible, no reembolsable y caduca el 30 de junio del curso correspondiente
  (usable todo el curso, incluido junio, en fechas abiertas por Ángela); se vende solo mientras
  haya fechas antes de esa caducidad que respeten una semana de antelación.
  *Negativo:* si se agota el tiempo sin fechas disponibles, no hay ampliación ni devolución
  automática; condiciones y caducidad exacta se muestran antes de comprar y en el vale.

- **[M-46] Cancelación con plazo de 48h** — Dado un taller o uso de regalo reservado, cuando el
  cliente cancela con 48 horas o más de antelación (hora exacta), entonces puede solicitar otra
  fecha disponible o devolución; con menos de 48h no hay devolución ni cambio garantizados salvo
  excepción de Ángela.

- **[M-47] Cancelación por parte de Ángela** — Dado que Ángela cancela un taller, cuando ocurre,
  entonces se envía correo para elegir otra fecha (requiere su aprobación) o solicitar reembolso
  (requiere su confirmación antes de ejecutarse); si la reserva se pagó con regalo, se restituye el
  uso del regalo conservando su caducidad, no dinero.

**Panel administrador**

- **[M-48] Gestión de turnos y aforo** — Dada la administradora, cuando crea, edita o cierra un
  turno, entonces el cambio se refleja de inmediato en lo que ven las alumnas al reservar.

- **[M-49] Vista de alumnas, pagos y créditos** — Dada la administradora, cuando abre el panel,
  entonces ve cada alumna con su turno, tipo de plan, estado del último pago y saldo de
  recuperaciones con sus caducidades.

- **[M-50] Registro manual de pagos externos** — Dado un pago por transferencia o efectivo, cuando
  Ángela lo recibe, entonces lo registra en la app (concepto, importe, fecha, método) y el sistema
  lo trata igual que un pago Stripe a efectos de habilitar clases y generar el borrador de factura.

### SHOULD

*(vacío por ahora — todo lo acordado con Ángela para el lanzamiento es MUST; lo que queda fuera
de esta versión está en "Fuera de alcance", no aquí, porque son cosas explícitamente pospuestas,
no funcionalidades secundarias del MVP.)*

### COULD

*(vacío — ver "Decisiones pendientes de aprobación" para lo que podría añadirse si Ángela lo
confirma.)*

### WON'T (esta versión)

- Notificaciones push móviles — fase posterior; v1 usa correo y avisos dentro de la app.
- Renovación automática de cuotas (cargo recurrente sin intervención) — fase posterior, aunque se
  use Stripe para el checkout puntual.
- Diseño de tarjetas de regalo personalizadas, dedicatorias y envíos programados — v1 solo cubre
  compra y uso básico del regalo.
- Soporte multi-tenant (varios estudios) — Kasa Juana es un solo estudio.

---

## Flujos de usuario principales

**Alta y primer pago:** Ángela da de alta a la alumna con su turno; la alumna completa perfil,
paga (o Ángela registra un pago externo) y se habilita la gestión de sus clases.

**Semana normal:** la alumna ve su turno confirmado por defecto. Si no puede ir, libera su plaza;
esa ausencia genera una recuperación. Otras alumnas con recuperaciones disponibles pueden ocupar
huecos Ceramiga (domingo 20:00) o huecos de titulares liberados (desde las 08:00 del día).

**Cancelación de Ángela:** si cancela una clase, las titulares reciben crédito; puede ofrecer una
sesión alternativa donde las afectadas tienen prioridad pero deben confirmar con un crédito propio.

**Pago mensual:** la alumna paga entre el día 1 y 7 (Stripe, transferencia o efectivo). Si no paga,
no se confirman las semanas siguientes hasta que lo haga; Ángela decide caso por caso si deja
asistir sin pagar.

**Renovación de curso:** del 15 al 30 de junio, las alumnas actuales reservan su plaza para el
curso siguiente con una señal de 10€. Las vacantes que queden se ofrecen a las interesadas
registradas, por orden de pago.

**Taller privado:** la responsable solicita fecha y grupo; Ángela aprueba en 48h; el pago en 24h
confirma la reserva; todo debe quedar cerrado (nombres, alergias, pagos) 24h antes del taller.

**Regalo:** se compra sin fecha; quien lo recibe la solicita cuando quiere, dentro del curso, antes
del 30 de junio, y Ángela aprueba.

---

## Requisitos no funcionales

- **Bloqueo de concurrencia en reservas y pagos:** al reservar una vacante (interesadas, Ceramiga,
  huecos liberados) el hueco debe quedar bloqueado brevemente durante el proceso de pago para que
  dos personas no puedan ocupar la misma plaza (plazo exacto: ver "Detalles pendientes de
  concretar").
- **Ningún movimiento de dinero sin aprobación humana:** reembolsos y ajustes de saldo los propone
  la app pero los ejecuta solo Ángela (M-28). Esto es una restricción de diseño, no solo de
  proceso: no debe existir una ruta que devuelva dinero sin su confirmación explícita.
- **Protección de datos:** datos de facturación y alimentarios visibles solo para la propia alumna
  y para Ángela (RGPD); el resto de datos de perfil (foto, nombre) son visibles entre alumnas del
  mismo turno.
- **Seguridad de pagos:** los datos de tarjeta de los pagos por Stripe no pasan ni se almacenan en
  la app (PCI DSS delegado a Stripe).
- **PWA instalable:** manifest + service worker; consulta de turno y saldo de recuperaciones
  usable con conectividad intermitente. Reservar y pagar requieren conexión.

---

## Decisiones pendientes de aprobación (Ángela)

Estas son propuestas que Ángela todavía no ha confirmado. No se implementan hasta que las apruebe
explícitamente; no se debe asumir ninguna resolución por defecto.

1. **Ajuste económico por mantenimiento dentro de un trimestre ya pagado.** Propuesta: mantener el
   coste de mantenimiento (35€) y generar un ajuste a favor por la diferencia con el valor
   proporcional del mes (ej. 178€ ÷ 3 − 35€ ≈ 24,33€), aplicado como descuento en el pago que cubra
   el mes inmediatamente posterior al último pagado, sin aplazarlo ni acumularlo. Sustituiría la
   compensación con clases actual.
2. **Facturación y gestoría.** Falta confirmar qué programa usa Ángela y cómo deben enviarse o
   integrarse las facturas ya aprobadas (M-29 genera el borrador; el destino final está abierto).
3. **Regalo afectado por cancelación de Ángela sin fechas disponibles antes del 30 de junio.**
   Propuesta: permitir que ella autorice una ampliación excepcional hasta una nueva fecha acordada.

## Detalles pendientes de concretar antes de lanzar

Ya acordados en principio; falta fijar el valor o mecanismo exacto antes de implementarlos:

- Cuándo reabrir la venta de regalos para el curso siguiente, y cómo identificar la caducidad de
  los que se compran en verano.
- Qué hacer con un ajuste económico ya aprobado si la alumna no realiza el siguiente pago.
- Plazo exacto del bloqueo breve durante el pago de una plaza, y permisos concretos para editar
  pagos y reservas (qué puede tocar cada rol).
- Margen de tiempo entre talleres privados para recoger y preparar el espacio.
- Campos exactos y permisos de la información alimentaria y la foto de perfil (marcadas como
  opcionales) antes de lanzar.

---

## Fuera de alcance (explícito)

- Multi-tenant (varios estudios o profesoras) — ver WON'T.
- Migración de histórico de pagos anteriores a Stripe/la app más allá de lo que Ángela cargue
  manualmente al lanzar (alumnas, turnos, pagos, recuperaciones, regalos existentes).
- Facturación con validez fiscal más allá de lo que ofrezca la herramienta elegida — el punto 2 de
  "Decisiones pendientes" define el alcance real cuando se cierre.
- Renovación automática de cuotas y notificaciones push — pospuestas explícitamente (ver WON'T).
