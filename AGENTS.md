# facturar — Facturacion electronica para Monotributo argentino (Codex)

Wrapper de lenguaje natural sobre [PyARCA](https://github.com/GeraCollante/PyARCA). Cuando el usuario menciona emitir una factura, nota de credito, consultar comprobantes o cualquier accion relacionada con facturacion electronica en Argentina, seguir este archivo.

---

## Paso 0 — Auto-deteccion y setup

Antes de cualquier accion, verificar PyARCA:

```bash
for dir in "${PYARCA_DIR:-}" ~/PyARCA ./PyARCA /opt/PyARCA; do
  [ -n "$dir" ] && [ -f "$dir/facturar.py" ] && echo "FOUND:$dir" && break
done
```

### Si PyARCA no esta instalado

```
PyARCA no encontrado. Puedo instalarlo ahora en ~/PyARCA. ¿Continuo?
```

Si el usuario acepta:

```bash
git clone https://github.com/GeraCollante/PyARCA ~/PyARCA
cd ~/PyARCA
uv sync
echo "export PYARCA_DIR=~/PyARCA" >> ~/.bashrc
export PYARCA_DIR=~/PyARCA
```

Si `uv` no esta:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.cargo/env
```

### Si `.env` falta o esta incompleto

```bash
cd $PYARCA_DIR
[ -f .env ] && grep -c "=" .env || echo "0"
```

Si tiene menos de 5 campos, lanzar wizard (ver seccion **Auto-configuracion**).

---

## Auto-configuracion — wizard de .env

```bash
cd $PYARCA_DIR
cp -n .env.example .env 2>/dev/null || true
```

| Campo | Que es | Ejemplo |
|-------|--------|---------|
| `CUIT` | Tu CUIT sin guiones | `20123456789` |
| `EMPRESA` | Nombre completo en ARCA | `JUAN PABLO PEREZ` |
| `MEMBRETE1` | Direccion | `Av. Corrientes 1234, CABA` |
| `MEMBRETE2` | CP y provincia | `C.P. 1043 - Buenos Aires` |
| `CUIT_FMT` | CUIT con guiones | `CUIT 20-12345678-9` |
| `IIBB` | Ingresos Brutos | `Exento` |
| `IVA` | Condicion IVA | `Responsable Monotributo` |
| `INICIO` | Inicio de actividad | `Inicio de Actividad: 01/01/2020` |
| `CERT` | Archivo .crt | `mi_certificado.crt` |
| `PRIVATEKEY` | Archivo .key | `mi_clave.key` |

Para `CERT` y `PRIVATEKEY`:

```
El certificado se obtiene en:
https://www.arca.gob.ar → Mis aplicaciones web → Administrador de Relaciones de Clave/Certificado

1. openssl genrsa -out mi_clave.key 4096
2. openssl req -new -key mi_clave.key -out mi_csr.csr
3. Subir el CSR en ARCA, descargar el .crt
4. Copiar ambos archivos a ~/PyARCA/
```

Escribir cada campo:

```bash
sed -i "s|^CUIT=.*|CUIT=$CUIT_VALOR|" $PYARCA_DIR/.env
```

Verificar al terminar:

```bash
cd $PYARCA_DIR && uv run python facturar.py listar 2>&1 | head -5
```

---

## Triggers — cuando aplicar estas instrucciones

Aplicar cuando el usuario dice (en cualquier forma):

- "emitir factura", "facturar", "hacer una factura", "necesito una factura"
- "nota de credito", "anular factura"
- "factura de exportacion", "factura E", "facturar al exterior"
- "consultar comprobante", "ver mis facturas", "listar facturas"
- cualquier mencion de ARCA, PyARCA, CAE, factura C, factura E en contexto de emision

---

## Comandos

### Emitir Factura C (mercado interno)

Datos necesarios:

| Campo | Requerido | Default |
|-------|-----------|---------|
| `--monto` | Si | — |
| `--cliente` | Si | `"Consumidor Final"` |
| `--descripcion` | Si | — |
| `--desde` | Si (AAAAMMDD) | — |
| `--hasta` | Si (AAAAMMDD) | — |
| `--cuit-cliente` | No | `0` |
| `--tipo-doc` | No | `99` (CF) / `80` (CUIT) / `96` (DNI) |
| `--punto-vta` | No | `3` |
| `--produccion` | Preguntar siempre | homologacion por default |

Confirmar antes de produccion:

```
Voy a emitir:
  Factura C — $[MONTO] — [CLIENTE]
  Periodo: [DESDE] → [HASTA]
  Ambiente: PRODUCCION ⚠️

¿Confirmas?
```

```bash
cd $PYARCA_DIR && uv run python facturar.py factura \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --descripcion "[DESCRIPCION]" \
  --desde [DESDE] \
  --hasta [HASTA] \
  [--cuit-cliente [CUIT]] \
  [--tipo-doc [TIPO]] \
  [--punto-vta [PTO]] \
  [--produccion]
```

Mostrar:

```
✅ Factura C emitida

Numero:      [N]
CAE:         [CAE]
Vencimiento: [FECHA]
Monto:       $[MONTO]
Cliente:     [CLIENTE]
Periodo:     [DESDE] → [HASTA]

Ambiente: [Produccion / Homologacion]
```

---

### Nota de Credito C

| Campo | Requerido |
|-------|-----------|
| `--monto` | Si |
| `--cliente` | Si |
| `--descripcion` | Si |
| `--desde` | Si |
| `--hasta` | Si |
| `--factura-asociada` | Si |
| `--cuit-cliente` | No |
| `--tipo-doc` | No |
| `--punto-vta` | No |
| `--produccion` | Preguntar |

```bash
cd $PYARCA_DIR && uv run python facturar.py nota-credito \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --descripcion "[DESCRIPCION]" \
  --desde [DESDE] \
  --hasta [HASTA] \
  --factura-asociada [NUMERO] \
  [--cuit-cliente [CUIT]] \
  [--tipo-doc [TIPO]] \
  [--punto-vta [PTO]] \
  [--produccion]
```

---

### Factura E (exportacion al exterior)

| Campo | Requerido | Default |
|-------|-----------|---------|
| `--monto` | Si | — |
| `--cliente` | Si | — |
| `--descripcion` | Si | — |
| `--pais-destino` | Si (codigo ARCA) | — |
| `--cuit-pais-cliente` | No | `""` |
| `--moneda` | No | `DOL` |
| `--tipo-cambio` | No | fetchear |
| `--incoterms` | No | `N/A` |
| `--idioma` | No | `7` |
| `--punto-vta` | No | `3` |
| `--produccion` | Preguntar | — |

Si el usuario no sabe el tipo de cambio, fetchearlo:

```bash
curl -s "https://dolarapi.com/v1/dolares/oficial" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['venta'])"
```

Codigos de pais comunes: 212=USA, 203=Brasil, 218=Mexico, 410=España, 438=Alemania, 426=UK, 412=Francia, 225=Uruguay, 208=Chile, 204=Canada.

```bash
cd $PYARCA_DIR && uv run python facturar.py factura-e \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --descripcion "[DESCRIPCION]" \
  --pais-destino [COD_PAIS] \
  [--cuit-pais-cliente [CUIT_PAIS]] \
  [--moneda [MONEDA]] \
  [--tipo-cambio [TC]] \
  [--incoterms [INC]] \
  [--idioma [IDIOMA]] \
  [--punto-vta [PTO]] \
  [--produccion]
```

---

### Nota de Credito E

```bash
cd $PYARCA_DIR && uv run python facturar.py nota-credito-e \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --descripcion "[DESCRIPCION]" \
  --pais-destino [COD_PAIS] \
  --factura-asociada [NUMERO] \
  [--cuit-pais-cliente [CUIT_PAIS]] \
  [--moneda [MONEDA]] \
  [--tipo-cambio [TC]] \
  [--produccion]
```

---

### Consultar comprobante

```bash
cd $PYARCA_DIR && uv run python facturar.py consultar \
  --numero [N] \
  [--nota-credito] \
  [--punto-vta [PTO]] \
  [--produccion]
```

```
📋 Comprobante N°[N]

Tipo:        [Factura C / NC C / Factura E / NC E]
CAE:         [CAE]
Vencimiento: [FECHA]
Monto:       $[MONTO]
Cliente:     [CLIENTE]
Estado:      [Vigente / Vencido]
```

---

### Listar comprobantes

```bash
cd $PYARCA_DIR && uv run python facturar.py listar
```

```
N°   Tipo        Fecha        Monto          Cliente              CAE
---  ----------  -----------  -------------  -------------------  ------------------
5    Factura C   2026-04-30   $1.500.000     Empresa XYZ SRL      12345678901234
```

---

## Manejo de errores

| Error | Accion |
|-------|--------|
| PyARCA no encontrado | Ofrecer auto-instalacion |
| `.env` incompleto | Lanzar wizard |
| Certificado invalido / vencido | Explicar renovacion en ARCA |
| ARCA timeout | Reintentar 1 vez; si falla, informar |
| Error desconocido | Mostrar stderr + auto-issue |

### Auto-issue

```bash
PYARCA_VER=$(cd ${PYARCA_DIR:-~/PyARCA} && git rev-parse --short HEAD 2>/dev/null || echo "desconocido")
OS_INFO=$(uname -s -r 2>/dev/null || echo "desconocido")

gh issue create \
  --repo kvothesson/facturar \
  --title "[auto] <titulo del problema>" \
  --body "## Que intentaba hacer el usuario

<descripcion>

## Problema encontrado

<error exacto>

## Contexto

- PyARCA commit: $PYARCA_VER
- OS: $OS_INFO
- Comando intentado: \`<comando>\`

## Stderr

\`\`\`
<stderr>
\`\`\`

---
*Issue generado automaticamente*"
```

---

## Tono

Directo y funcional. Confirmar siempre el ambiente antes de produccion. Mostrar CAE y vencimiento de forma prominente.

---

## Fuentes

- PyARCA: https://github.com/GeraCollante/PyARCA
- ARCA portal: https://www.arca.gob.ar
- Codigos de paises: https://www.afip.gob.ar/fe/documentos/paises.xls
