Eres un programador experto full-stack especializado en React, JavaScript moderno y desarrollo web. Tu rol es ayudar a mejorar una aplicación web existente (CRM de exportaciones llamado "Antigravity Export CRM") de manera sistemática, práctica y didáctica.

## Tu Personalidad y Enfoque

1. **Didáctico**: Explicas todo paso a paso, como si le enseñaras a alguien que está aprendiendo. Usas analogías simples cuando sea útil.
2. **Práctico**: Proporcionas código funcional y listo para usar. Nada de ejemplos incompletos.
3. **Estructurado**: Analizas antes de sugerir. Presentas opciones cuando hay múltiples caminos.
4. **Respetuoso del contexto**: Usas el archivo `claude.md` en la raíz del proyecto como referencia para mantener consistencia con el código existente.

## Proceso de Trabajo

Cuando el usuario pida mejoras, sigue este flujo:

### PASO 1: Verificar Contexto
- Busca y lee el archivo `claude.md` si existe en el proyecto
- Si no existe, créalo basándote en el análisis del código

### PASO 2: Análisis del Código
Antes de sugerir cambios, evalúa:
- Arquitectura: ¿Cómo está organizado? ¿Qué patrones usa?
- Dependencias: ¿Qué librerías utiliza?
- Estilo: ¿Qué convenciones de código sigue?
- Puntos fuertes y áreas de mejora

### PASO 3: Propuesta de Mejoras
Presenta las mejoras organizadas por prioridad:
- 🔴 Alta: Bugs críticos, Seguridad
- 🟠 Media: Rendimiento
- 🟡 Normal: UX/UI
- 🟢 Baja: Código limpio, refactorizaciones

### PASO 4: Implementación
Para cada mejora:
1. Explica QUÉ vas a cambiar y POR QUÉ
2. Implementa el código directamente en los archivos
3. Explica cómo verificar que funciona

### PASO 5: Documentación
Después de implementar cambios, actualiza el archivo `claude.md` con:
- Qué se modificó
- Fecha del cambio
- Notas relevantes

## Áreas de Especialidad

Cuando analices código web, considera:
- **Rendimiento**: Lazy loading, memoización, optimización de re-renders
- **Seguridad**: Validación de inputs, manejo seguro de credenciales
- **Accesibilidad**: Semántica HTML, labels, contraste, navegación por teclado
- **UX**: Feedback visual, manejo de errores, diseño responsive
- **Mantenibilidad**: Separación de concerns, componentes reutilizables, naming claro

## Reglas Importantes

1. NO asumas tecnologías - Pregunta si no estás seguro qué frameworks usa el proyecto
2. NO hagas cambios masivos - Prefiere mejoras incrementales
3. NO ignores el contexto - Respeta el `claude.md` como fuente de verdad
4. SÍ explica el "por qué" - El usuario quiere aprender, no solo tener código
5. SÍ ofrece alternativas con pros/contras cuando hay múltiples soluciones
6. SÍ valida antes de implementar cambios grandes

## Contexto del Proyecto Actual

El proyecto es un CRM single-file HTML con:
- React 18 (vía CDN)
- Supabase (DB + Auth + Storage)
- Chart.js, Flatpickr, SheetJS, PDF.js
- localStorage como respaldo offline

Funcionalidades: Dashboard, CRUD de viajes, Kanban, Calendario, Gestión de PDFs, Import/Export Excel, Notificaciones, Modo oscuro.

---

## Historial de Cambios

### 2026-01-23 - Sesión de mejoras

#### 🔴 Correcciones Críticas

**1. Fechas con día atrasado (Timezone Issue)**
- **Problema:** Las fechas mostraban un día menos tanto en exportación Excel como en la tabla de viajes
- **Causa:** JavaScript interpreta fechas ISO (YYYY-MM-DD) como UTC medianoche, y al convertir a hora local (México UTC-6) restaba un día
- **Solución en exportación Excel:**
  - Función `formatDateExport` ahora suma 1 día para compensar
  - Formato cambiado a DD/MM/YYYY
  - Opción `raw: true` en `json_to_sheet` para evitar auto-conversión
- **Solución en visualización:**
  - Función `robustParseDate` ahora parsea fechas ISO manualmente sin UTC
  - Detecta formato YYYY-MM-DD y crea Date con hora local
- **Archivos:** `pagina_metales_mejorada.html` (líneas ~1335-1380, ~1413-1460)

**2. Error React/Lucide al subir documentos**
- **Problema:** Error `NotFoundError: Failed to execute 'removeChild'` al subir PDFs
- **Causa:** Lucide.js reemplaza elementos `<i>` con SVGs, React no encuentra los nodos originales al actualizar
- **Solución:**
  - Estructura DOM fija (ambas vistas siempre existen, se muestran/ocultan con CSS)
  - Eliminé iconos de Lucide en partes dinámicas (usé emoji 📄 en lugar de icono Upload)
  - Estado local `localDoc` para control independiente
- **Archivo:** `pagina_metales_mejorada.html` (componente `DocumentCard` ~línea 2349)

#### 🟠 Mejoras de Funcionalidad

**3. Supabase Storage para documentos PDF**
- **Cambio:** Los PDFs ahora se suben a Supabase Storage en lugar de Base64 en localStorage
- **Beneficio:** Sin límite de 5MB de localStorage, archivos hasta 50MB
- **Implementación:**
  - `StorageService` usa `fetch` directo a la API (no requiere librería Supabase JS)
  - Bucket: `documentos` (público)
  - Estructura: `documentos/{tripId}/{docTypeId}_{timestamp}_{filename}.pdf`
- **Auto-guardado:** Al subir documento, el viaje se guarda automáticamente
- **Configuración Supabase:**
  - Bucket `documentos` creado y público
  - Políticas: INSERT y SELECT para rol `anon`

**4. Carpeta de documentos clickeable**
- **Cambio:** Al hacer clic en el indicador de documentos (📁 1/6) en la tabla de viajes, se abre el modal directamente en la pestaña de Documentos
- **Implementación:**
  - Nueva función `openEditDocs(viaje)` que abre modal con `initialTab='documentos'`
  - Prop `initialTab` agregada a `EditViajeModal`
  - Efecto hover en el indicador para mostrar que es clickeable

**5. Campo Análisis reubicado**
- **Antes:** Análisis estaba en la sección de Finanzas
- **Ahora:** Análisis (%) está en la sección de "Carga" junto con Contenedores y Peso
- **Razón:** El análisis es un dato de la carga, no financiero

**6. Venta en USD (no MXN)**
- **Cambio:** "Venta Total (MXN)" → "Venta Total (USD)"
- **Cambio:** "Saldo Pendiente (MXN)" → "Saldo Pendiente (USD)"
- **Resumen:** Ahora muestra Venta (USD), Por Cobrar (USD), Flete (USD)

#### 🟢 Mantenimiento

**7. Limpieza de archivos duplicados**
- Se eliminaron 5 PDFs duplicados de prueba en Supabase Storage
- Se eliminó archivo test.txt de prueba

**8. Backup creado**
- Archivo: `pagina_metales_mejorada_backup_2026-01-22.html`
- Ubicación: Mismo directorio del proyecto

---

## Configuración Actual

### Supabase (Base de Datos + Storage)
- **URL:** `https://cteczvfjnkhipbvbwlmx.supabase.co`
- **Database:** Tabla `viajes` con Row Level Security (RLS)
- **Storage:** Buckets `documentos` y `backups` (públicos)
- **Auth:** Sistema de perfiles con roles (admin, editor_restringido)

### Estructura de Documentos por Viaje
```javascript
DOCUMENT_TYPES = [
  { id: 'booking', label: 'Booking' },
  { id: 'bl', label: 'BL' },
  { id: 'analisis', label: 'Análisis' },
  { id: 'packing', label: 'Packing List' },
  { id: 'anticipo', label: 'Comprobante de Anticipo' },
  { id: 'analisisCliente', label: 'Análisis Cliente' }
]
```

---

### 2026-01-27 - Historial de Cambios

#### 🟢 Nueva Funcionalidad

**11. Historial de cambios por viaje**
- **Cambio:** Cada viaje ahora guarda un registro de todas las modificaciones
- **Funcionalidad:**
  - Nuevo tab "Historial" en el modal de edicion (5to tab)
  - Registra: fecha/hora, usuario, accion (Creado/Modificado/Documento subido/eliminado)
  - Muestra campos modificados con valor anterior y nuevo
  - Colores por tipo de accion: verde (creado), azul (doc subido), rojo (doc eliminado), amarillo (modificado)
- **Estructura de datos:**
```javascript
viaje.historial = [
  {
    id: 'uuid',
    fecha: '2026-01-27T10:30:00.000Z',
    usuario: 'usuario@email.com',
    accion: 'Modificado',
    cambios: {
      'Status': { antes: 'Programacion', despues: 'Ingresados' },
      'Contenedores': { antes: '2', despues: '3' }
    }
  }
]
```
- **Campos rastreados:** Nombre, Booking, BL, Fecha, Embarque, Arribo, Status, SI, Cut Off, Contenedores, Peso, Analisis, Flete, Venta, Deben, 10%, Anticipos, Costo Agencia, Agencia Aduanal, Origen, Destino, Cliente, Proveedor, Producto, Notas
- **Lineas modificadas:** ~3187-3250 (funciones historial), ~3269-3340 (tabs), ~3597-3710 (UI historial), ~5417-5480 (handleSave)

**12. Validación de campos numéricos**
- **Cambio:** Los campos numéricos ahora validan en tiempo real que solo contengan números
- **Campos validados:**
  - Enteros: Contenedores
  - Decimales: Peso, Análisis, Flete, Costo agencia, $venta, $deben, 10%, Anticipos
- **Comportamiento:**
  - Limpia automáticamente caracteres no válidos
  - Muestra borde rojo y mensaje de error si hay problema
  - No permite guardar si hay errores de validación
- **Bug corregido:** El textarea de "Observaciones" usaba el campo Analisis - ahora usa el campo Notas
- **Líneas modificadas:** ~3251-3320 (funciones validación), ~3350-3390 (set con validación), ~457-480 (CSS errores)

### 2026-01-27 - Página de Historial y Sistema de Anticipos

#### 🟠 Mejoras de Funcionalidad

**13. Página de Historial independiente**
- **Cambio:** El historial ahora tiene su propia página en el sidebar (en lugar de estar dentro del modal)
- **Beneficio:** Se puede ver el historial de TODOS los viajes en un solo lugar
- **Funcionalidad:**
  - Nueva página "Historial" accesible desde el sidebar
  - Lista todos los cambios de todos los viajes ordenados por fecha (más reciente primero)
  - Filtro para buscar por nombre de viaje
  - Click en cualquier cambio abre el modal de ese viaje
- **Líneas modificadas:** ~2512 (sidebar), ~5267-5410 (HistorialPage)

**14. Sistema de Anticipos mejorado**
- **Cambio:** Nuevo campo "Anticipo Pagado" (checkbox) para marcar cuando ya te pagaron el anticipo
- **Funcionalidad:**
  - Checkbox en el tab Finanzas del modal de edición
  - Color verde cuando está pagado, naranja cuando está pendiente
  - En la tabla de Viajes: muestra ✓ y color verde si pagado, naranja si pendiente
  - Saldo Pendiente ahora solo cuenta anticipos NO pagados
- **KPIs actualizados en Dashboard:**
  - "Anticipos Pendientes" (naranja) - suma de anticipos no pagados
  - "Anticipos Pagados" (verde) - suma de anticipos ya cobrados
  - "Saldo Pendiente" = Anticipos Pendientes + 10% Cobrable
- **Historial:** El campo se rastrea y muestra "Pagado"/"Pendiente" en lugar de true/false
- **Estructura de datos:**
```javascript
viaje.anticipoPagado = true // o false
```
- **Líneas modificadas:** ~3213-3248 (TRACKED_FIELDS, FIELD_LABELS), ~3340-3355 (formatChangeValue), ~3631-3645 (checkbox UI), ~3768-3810 (stats y KPIs), ~4028 (tabla display)

**15. Manejo de errores en sincronización**
- **Problema:** Si fallaba Supabase durante guardado/eliminación/importación, la UI se congelaba porque `setSyncing(false)` nunca se ejecutaba
- **Solución:** Agregado try/catch/finally a las funciones principales:
  - `handleSave`: Si falla, muestra error pero siempre desbloquea la UI
  - `handleDelete`: Igual, con manejo separado para Supabase
  - `handleImport`: Igual, notifica si la importación local funcionó pero Supabase falló
- **Patrón usado:**
  - Guardar localmente primero (localStorage)
  - Luego intentar sincronizar con Supabase (si falla, no bloquea)
  - `finally` siempre ejecuta `setSyncing(false)` para desbloquear UI
- **Líneas modificadas:** ~5637-5716 (handleSave), ~5718-5739 (handleDelete), ~5741-5763 (handleImport)

**16. Fix pérdida de foco en inputs numéricos**
- **Problema:** Al escribir en campos numéricos (Anticipos, Análisis, etc.), el cursor perdía foco después de cada tecla
- **Causa:** El componente `FieldWithChange` estaba definido DENTRO de `EditViajeModal`, causando que React lo recreara en cada render
- **Solución:** Movido `FieldWithChange` fuera del modal como componente independiente. Ahora recibe props: `isNew`, `original`, `form`, `errors` via `{...fieldProps}`
- **Líneas modificadas:** ~3355-3378 (FieldWithChange externo), ~3414 (fieldProps)

**17. Confirmación al cerrar modal con cambios**
- **Problema:** Si el usuario hacía cambios y cerraba el modal sin guardar, perdía todo sin aviso
- **Solución:**
  - Nueva función `hasUnsavedChanges()` compara el formulario actual con el viaje original
  - Nueva función `handleClose()` pregunta "¿Descartar cambios sin guardar?" si hay cambios
  - Aplica a: click en overlay, botón X, botón Cancelar
- **Campos comparados:** Nombre, Booking, BL, fechas, Status, SI, Cut off, Contenedores, Peso, Análisis, datos financieros, ubicaciones, participantes, Notas
- **Líneas modificadas:** ~3416-3445 (hasUnsavedChanges, handleClose), ~3493-3497 (overlay y botón X), ~3787 (botón Cancelar)

**18. Debounce en búsqueda**
- **Problema:** La tabla se re-renderizaba con cada tecla al escribir en el buscador, causando parpadeo
- **Solución:**
  - Nuevo estado `searchInput` para el valor inmediato del input
  - `useEffect` con `setTimeout` de 300ms actualiza `search` (el valor real para filtrar)
  - El usuario ve su texto inmediatamente, pero la tabla solo se filtra 300ms después de dejar de escribir
- **Líneas modificadas:** ~3934-3947 (estados y useEffect), ~4036 (input)

**19. Tecla ESC cierra modal**
- **Mejora:** Presionar Escape cierra el modal de edición (con confirmación si hay cambios)
- **Implementación:** `useEffect` con event listener para `keydown` que llama a `handleClose`
- **Líneas modificadas:** ~3447-3456 (useEffect para ESC)

---

### 2026-01-30 - Fix Sincronización Supabase

#### 🔴 Corrección Crítica

**20. URL de librería Supabase corregida (404 → 200)**
- **Problema:** Los viajes creados en un dispositivo NO aparecían en otros dispositivos
- **Causa raíz:** La URL de la librería Supabase apuntaba a `supabase.min.js` que no existe (404)
- **Consecuencia:** La librería nunca se cargaba, `window.supabase` era `undefined`, todos los datos se guardaban solo en localStorage (local al dispositivo)
- **Solución:** Cambiar URL a `supabase.js` (sin `.min`)
- **Archivo:** `index.html` (línea 18)

**21. Indicador visual de conexión Supabase**
- **Cambio:** Nuevo indicador en el header que muestra "Nube" (verde) o "Offline" (rojo)
- **Beneficio:** El usuario puede ver inmediatamente si la sincronización está funcionando
- **Implementación:** Estado `supabaseConnected` + indicador visual en Header

**22. Función waitForSupabase()**
- **Cambio:** Nueva función que espera hasta 5 segundos a que la librería Supabase esté disponible
- **Uso:** Se llama antes de `getViajes()` y `saveViaje()` para asegurar que Supabase esté listo
- **Archivo:** `index.html` (líneas ~1192-1222)

**23. Logging mejorado en saveViaje**
- **Cambio:** Logs detallados que muestran exactamente qué pasa cuando se guarda un viaje
- **Beneficio:** Facilita depuración si hay problemas de sincronización

### 2026-02-01 - Limpieza de Código

#### 🟢 Mantenimiento

**24. Eliminación de código no utilizado**
- **Cambio:** Limpieza de código muerto que nunca se ejecutaba
- **Elementos eliminados:**
  - `AlertasPage` - Componente nunca renderizado
  - `RealtimeService` - Servicio nunca invocado
  - `ProfileService.updateProfile()`, `getAllProfiles()`, `isAdmin()` - Métodos no usados
  - `SupabaseAuthService.onAuthStateChange()` - Método no llamado
  - `CAMPOS_FINANCIEROS_RESTRINGIDOS` - Variable no usada
  - `EvidenceIcon` - Componente no integrado
- **Total:** 162 líneas eliminadas sin afectar funcionalidad

**25. Filtro multi-select de status en Viajes**
- **Cambio:** Ahora se pueden seleccionar múltiples status para filtrar (ej: Altamar + Ingresados)
- **Implementación:** Dropdown con checkboxes en lugar de select simple
- **UI:** Badges que muestran filtros activos con opción de remover individualmente

**26. Selector de fechas visible en móvil**
- **Cambio:** El date picker ahora es visible en dispositivos móviles
- **Antes:** Oculto con `display: none` en pantallas pequeñas

**27. Documentación actualizada**
- **Cambio:** Eliminadas referencias a Google Sheets (ya no se usa)
- **Estado actual:** Solo Supabase como backend (DB + Storage + Auth)

---

### 2026-02-20 - Auditoría completa + Fix de fechas

#### 🔴 Corrección Crítica

**28. Fix fechas invertidas en extracción de Booking PDF**
- **Problema:** Las fechas se extraían con día y mes al revés (asumía MM/DD/YYYY americano)
- **Causa:** `convertEnglishDateToISO` asumía formato americano, pero navieras usan DD/MM/YYYY
- **Solución:** Renombrada a `convertDateToISO`, ahora asume DD/MM/YYYY por defecto
- **Efecto:** También corrige el nombre del viaje (ZN DD-MM-YY) que se generaba mal

#### 🟠 Bugs encontrados en auditoría

**29. Búsqueda expandida a más campos**
- **Antes:** Solo buscaba en Nombre, Booking y BL
- **Ahora:** También busca en Cliente, Proveedor, Destino, Origen y Producto

**30. Excel Import preserva historial y createdAt**
- **Problema:** Importar Excel sobre viajes existentes sobreescribía historial de cambios
- **Solución:** Se preservan `historial`, `documentos` y `createdAt` al actualizar

**31. Export Excel incluye campos faltantes**
- **Agregados a EXPORT_COLUMNS:** `SI`, `anticipoPagado`, `fechaAnticipoPagado`

**32. SortHeader movido fuera de ViajesPage**
- **Problema:** Componente definido dentro de otro (anti-patrón React, causaba re-mount)
- **Solución:** Movido como componente externo con props `sortConfig` y `onSort`

**33. ESC listener optimizado con useRef**
- **Problema:** useEffect con deps `[form, viaje]` re-registraba listener en cada tecla
- **Solución:** useRef para handleClose + dependency array vacía `[]`

**34. detectChanges corregido para campos numéricos**
- **Problema:** `0` vs `''` generaba falso positivo en historial de cambios
- **Solución:** Campos numéricos se comparan como números (0 === '' → sin cambio)

**35. Historial null check robusto**
- **Antes:** `if (!toSave.historial)` — no cubre caso `null`
- **Ahora:** `if (!Array.isArray(toSave.historial))` — cubre null, undefined, string, etc.

**36. Validación de negativos en campos físicos**
- **Campos afectados:** Contenedores, Peso, Análisis
- **Cambio:** No se permiten valores negativos, muestra error "No se permiten negativos"

**37. Redondeo financiero al guardar**
- **Campos:** Flete, $venta, $deben, 10%, Anticipos, Agencia aduanal
- **Cambio:** Se redondean a 2 decimales antes de guardar para evitar errores de punto flotante

---

### 2026-07-08 - SI/VGM manual + Cierre Despacho automático

#### 🟠 Cambio de lógica de fechas

**38. SI/VGM ahora es manual; Cierre Despacho se auto-calcula (Zarpe − 3 días)**
- **Problema:** La regla de "3 días antes del zarpe" estaba aplicada al campo equivocado. SI/VGM se auto-calculaba (debe ser manual, viene del booking) y Cierre Despacho era manual (es el que va a 3 días del zarpe).
- **Solución:** Se invirtió la lógica:
  - **SI/VGM** (`Cut off`): 100% manual. Se quitó todo auto-cálculo.
  - **Cierre Despacho** (`Cierre despacho`): se auto-calcula = Zarpe − 3 días al cambiar el Zarpe, pero sigue siendo editable a mano y admite estado Verde/Rojo (Opción A). Si ya está en Verde/Rojo, el cambio de Zarpe no lo pisa.
- **Lugares modificados en `index.html`:**
  1. `onChange` del Zarpe en el modal (~5131): ahora setea `Cierre despacho` en vez de `Cut off`, salvo que esté en Verde/Rojo.
  2. Labels del modal (~5151, ~5160): "Fecha SI/VGM (manual)" y "Cierre Despacho (auto: 3 días antes de Zarpe)".
  3. `handleSave` al crear viaje nuevo (~7710): solo auto-calcula Cierre Despacho; SI/VGM se deja como venga (manual o del PDF).
  4. Migración `migration_sivgm_cierre_v2` (~7544): se eliminó el Paso 2 que forzaba SI/VGM = Zarpe − 3; se conserva el Paso 1 que copia el SI/VGM viejo al Cierre Despacho.
- **Nota:** Los datos ya migrados no se re-tocan (la flag v2 ya está puesta). Los valores de SI/VGM existentes quedan como estaban y ahora son editables a mano.

**39. Limpieza de columnas en exportación Excel (espacios en blanco)**
- **Problema:** El Excel exportado tenía muchos espacios en blanco. Diagnóstico sobre los 65 viajes reales de Supabase.
- **Causa:** 5 columnas "fantasma" que se exportaban pero **no tienen campo en el formulario** para llenarse, así que salían 100% vacías en todos los viajes: `Origen`, `Destino`, `Cliente`, `Proveedor`, `Producto`. (El cliente se captura en el campo `Nombre`, etiquetado "Nombre / Cliente".)
- **Solución en `EXPORT_COLUMNS` (~línea 3487):**
  - Se **quitaron** las 5 columnas fantasma.
  - Se **quitó** la columna vieja `SI` (semáforo legacy), dejando solo `Cut off` (= SI/VGM real) y `Cierre despacho`.
  - Se **agregó** `Costo agencia` (tenía datos en 3 viajes pero no se exportaba).
- **Resultado:** El Excel pasó de 29 a 24 columnas, sin ninguna columna 100% vacía.
- **No se tocó:** `TRACKED_FIELDS`, `FIELD_LABELS` (historial) ni el mapeo de importación — siguen reconociendo esos nombres por compatibilidad.
- **Nota:** Las columnas que quedan vacías "a veces" (Nº factura, 10%, Anticipos, Notas, fechas ETD/ETA) son normales: solo se llenan cuando el viaje tiene ese dato.

---

## Pendientes / Mejoras Futuras

1. ~~**Validación de datos**~~ ✅ Implementado 2026-01-27
2. ~~**Historial de cambios**~~ ✅ Implementado 2026-01-27
3. ~~**Sistema de anticipos mejorado**~~ ✅ Implementado 2026-01-27
4. ~~**Manejo de errores en sincronización**~~ ✅ Implementado 2026-01-27
5. ~~**Confirmación al cerrar modal**~~ ✅ Implementado 2026-01-27
6. ~~**Debounce en búsqueda + ESC**~~ ✅ Implementado 2026-01-27
7. ~~**Migración a Supabase**~~ ✅ Implementado 2026-01-30
8. **Notificaciones inteligentes** - Alertas de fechas próximas (Cut off, Arribo)
9. **Migración de PDFs existentes** - Mover documentos Base64 antiguos a Supabase Storage
10. **Filtros avanzados** - Filtrar por Cliente, Proveedor, rango de fechas

---

## Para Comenzar

Al inicio de cada sesión:
1. Lee el archivo `claude.md` si existe
2. Pregunta qué quiere mejorar el usuario
3. Analiza el código relevante
4. Propón mejoras priorizadas
5. Implementa lo que el usuario apruebe
6. Actualiza la documentación
