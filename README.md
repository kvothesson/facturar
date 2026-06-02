# facturar

Facturacion electronica para Monotributo argentino — compatible con **Claude Code** y **OpenAI Codex**.

Wrapper de lenguaje natural sobre [PyARCA](https://github.com/GeraCollante/PyARCA). Le decis a Claude "emitir factura por $1.500.000 a Empresa XYZ por desarrollo de software abril 2026" y el instala lo que falta, configura lo que no esta configurado y emite el comprobante.

## Instalacion

### Claude Code

```bash
claude --plugin-dir /ruta/a/facturar
```

O instalar desde el [marketplace de ar-plugins](https://github.com/kvothesson/ar-plugins).

### OpenAI Codex

No requiere instalacion de plugin. Codex lee `AGENTS.md` automaticamente al abrirse en este directorio (o cualquier directorio padre).

```bash
git clone https://github.com/kvothesson/facturar ~/facturar
cd ~/facturar
codex   # AGENTS.md se carga automaticamente
```

Luego decirle en lenguaje natural: *"emitir factura por $1.500.000 a Empresa XYZ por desarrollo de software abril 2026"*.

## Setup automatico

La primera vez que uses el plugin, Claude detecta si PyARCA esta instalado y si el `.env` esta configurado. Si no, lo resuelve solo:

**PyARCA no instalado:**
```
PyARCA no encontrado. Puedo instalarlo ahora en ~/PyARCA. ¿Continuo?
> si
Clonando PyARCA... instalando dependencias... listo.
```

**`.env` no configurado:**
```
Necesito configurar tus datos fiscales (se hace una sola vez).

CUIT (sin guiones): 20123456789
Nombre completo en ARCA: JUAN PABLO PEREZ
Direccion: Av. Corrientes 1234, CABA
...
```

Para el certificado digital (se obtiene en ARCA una sola vez), Claude te guia paso a paso.

## Comandos y ejemplos

> En Claude Code se usan como slash commands. En Codex se describen en lenguaje natural — el agente los detecta y ejecuta igual.

### Factura C — `/facturar:facturar factura` (Claude Code) / "emitir factura" (Codex)

Emitir Factura C (mercado interno).

```
✅ Factura C emitida

Numero:      5
CAE:         12345678901234
Vencimiento: 2026-05-17
Monto:       $1.500.000
Cliente:     Empresa XYZ SRL
Periodo:     2026-04-01 → 2026-04-30

Ambiente: Produccion
```

---

### Nota de Credito C — `/facturar:facturar nota-credito` (Claude Code) / "nota de credito" (Codex)

Anular una Factura C con Nota de Credito C.

```
✅ Nota de Credito C emitida

Numero:           2
CAE:              12345678901235
Vencimiento:      2026-05-17
Monto:            $1.500.000
Factura asociada: N° 5
Cliente:          Empresa XYZ SRL

Ambiente: Produccion
```

---

### Factura E — `/facturar:facturar factura-e` (Claude Code) / "factura de exportacion" (Codex)

Emitir Factura E (exportacion de servicios al exterior). Claude fetchea el tipo de cambio oficial automaticamente si no lo indicás.

```
✅ Factura E emitida

Numero:      3
CAE:         12345678901236
Vencimiento: 2026-05-17
Monto:       u$s 1.200 (TC: $1.410 → $1.692.000 ARS)
Cliente:     Acme Corp
Pais:        USA (200)

Ambiente: Produccion
```

---

### Consultar — `/facturar:facturar consultar` (Claude Code) / "consultar comprobante N°5" (Codex)

Consultar un comprobante especifico.

```
📋 Comprobante N°5

Tipo:        Factura C
CAE:         12345678901234
Vencimiento: 2026-05-17
Monto:       $1.500.000
Cliente:     Empresa XYZ SRL
Estado:      Vigente
```

---

### Listar — `/facturar:facturar listar` (Claude Code) / "listar mis facturas" (Codex)

Listar todos los comprobantes emitidos.

```
N°   Tipo        Fecha        Monto          Cliente              CAE
---  ----------  -----------  -------------  -------------------  ------------------
5    Factura C   2026-04-30   $1.500.000     Empresa XYZ SRL      12345678901234
4    Factura C   2026-03-31   $1.200.000     Consumidor Final     12345678901233
3    Factura E   2026-02-28   u$s 1.200      Acme Corp            12345678901232
```

---

## Auto-issue

Si el plugin encuentra un error que no sabe resolver o una funcionalidad no cubierta, crea un issue en este repo automaticamente con el contexto completo para que sea fixeado.

## Fuentes

- PyARCA: https://github.com/GeraCollante/PyARCA
- ARCA portal: https://www.arca.gob.ar
- Discusion original: https://www.reddit.com/r/merval/comments/1t5ivpv/comment/okdmyvd/
