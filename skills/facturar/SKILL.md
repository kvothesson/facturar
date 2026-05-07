---
description: Facturacion electronica para Monotributo argentino via PyARCA. Emite Facturas C, NC C, Facturas E y NC E contra ARCA desde lenguaje natural. Se instala y configura solo. Usalo con /facturar:facturar factura, nota-credito, factura-e, nota-credito-e, consultar, listar.
---

# Skill: /facturar:facturar

Wrapper de lenguaje natural sobre [PyARCA](https://github.com/GeraCollante/PyARCA). Traduce lo que el usuario describe a comandos PyARCA y los ejecuta via Bash.

---

## Paso 0 — Auto-deteccion y setup

Antes de cualquier comando, ejecutar este bloque de deteccion:

```bash
# Buscar PyARCA en orden de prioridad
for dir in "${PYARCA_DIR:-}" ~/PyARCA ./PyARCA /opt/PyARCA; do
  [ -n "$dir" ] && [ -f "$dir/facturar.py" ] && echo "FOUND:$dir" && break
done
```

### Si PyARCA no esta instalado

Ofrecer instalarlo automaticamente:

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

Si `uv` no esta instalado:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.cargo/env
```

Luego retomar el sync.

### Si PyARCA esta instalado pero `.env` no existe o esta incompleto

Detectar:

```bash
cd $PYARCA_DIR
[ -f .env ] && grep -c "=" .env || echo "0"
```

Si `.env` no existe o tiene menos de 5 campos, lanzar el wizard de configuracion (ver seccion **Auto-configuracion**).

---

## Auto-configuracion — wizard de .env

Guiar al usuario campo por campo. Mostrar el campo, explicar que es, esperar la respuesta, escribirla al `.env`.

```bash
cd $PYARCA_DIR
cp -n .env.example .env 2>/dev/null || true
```

Campos a completar en orden:

| Campo | Que es | Ejemplo |
|-------|--------|---------|
| `CUIT` | Tu CUIT sin guiones | `20123456789` |
| `EMPRESA` | Tu nombre completo tal como figura en ARCA | `JUAN PABLO PEREZ` |
| `MEMBRETE1` | Direccion (aparece en la factura) | `Av. Corrientes 1234, CABA` |
| `MEMBRETE2` | CP y provincia | `C.P. 1043 - Buenos Aires` |
| `CUIT_FMT` | CUIT con guiones (para el PDF) | `CUIT 20-12345678-9` |
| `IIBB` | Situacion en Ingresos Brutos | `Exento` o numero de IIBB |
| `IVA` | Condicion frente al IVA | `Responsable Monotributo` |
| `INICIO` | Fecha de inicio de actividad | `Inicio de Actividad: 01/01/2020` |
| `CERT` | Nombre del archivo .crt | `mi_certificado.crt` |
| `PRIVATEKEY` | Nombre del archivo .key | `mi_clave.key` |

Para `CERT` y `PRIVATEKEY`, el certificado digital lo emite ARCA. Si el usuario no lo tiene:

```
El certificado digital se obtiene en:
https://www.arca.gob.ar → Mis aplicaciones web → Administrador de Relaciones de Clave/Certificado

Pasos:
1. Generar clave privada: openssl genrsa -out mi_clave.key 4096
2. Generar CSR: openssl req -new -key mi_clave.key -out mi_csr.csr
3. Subir el CSR en ARCA y descargar el .crt
4. Copiar ambos archivos a ~/PyARCA/

Avisa cuando lo tengas y continuamos.
```

Escribir cada campo al `.env`:

```bash
sed -i "s|^CUIT=.*|CUIT=$CUIT_VALOR|" $PYARCA_DIR/.env
```

Al terminar, verificar:

```bash
cd $PYARCA_DIR && uv run python facturar.py listar 2>&1 | head -5
```

Si responde sin error de certificado → configuracion ok. Si falla → mostrar el error exacto y diagnosticar.

---

## Comandos

### `/facturar:facturar factura`

Emitir una Factura C (mercado interno).

Datos que necesita — preguntar los que no esten claros en el mensaje del usuario:

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

Confirmar antes de ejecutar en produccion:

```
Voy a emitir:
  Factura C — $[MONTO] — [CLIENTE]
  Periodo: [DESDE] → [HASTA]
  Ambiente: PRODUCCION ⚠️

¿Confirmas?
```

Ejecutar:

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

Mostrar resultado:

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

### `/facturar:facturar nota-credito`

Anular o ajustar una Factura C con Nota de Credito C.

| Campo | Requerido | Default |
|-------|-----------|---------|
| `--monto` | Si | — |
| `--cliente` | Si | — |
| `--descripcion` | Si | — |
| `--desde` | Si (AAAAMMDD) | — |
| `--hasta` | Si (AAAAMMDD) | — |
| `--factura-asociada` | Si | — |
| `--cuit-cliente` | No | `0` |
| `--tipo-doc` | No | `99` |
| `--punto-vta` | No | `3` |
| `--produccion` | Preguntar siempre | — |

Confirmar antes de ejecutar. Ejecutar:

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

### `/facturar:facturar factura-e`

Emitir Factura E (exportacion de servicios al exterior).

| Campo | Requerido | Default |
|-------|-----------|---------|
| `--monto` | Si | — |
| `--cliente` | Si | — |
| `--descripcion` | Si | — |
| `--pais-destino` | Si (codigo ARCA) | — |
| `--cuit-pais-cliente` | No | `""` |
| `--moneda` | No | `DOL` |
| `--tipo-cambio` | No (sugerido fetchearlo) | — |
| `--incoterms` | No | `N/A` |
| `--idioma` | No | `7` (Español) |
| `--punto-vta` | No | `3` |
| `--produccion` | Preguntar siempre | — |

Si el usuario no sabe el tipo de cambio, fetchearlo:

```bash
curl -s "https://dolarapi.com/v1/dolares/oficial" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['venta'])"
```

Codigos de pais comunes: 200=USA, 101=Alemania, 105=Brasil, 115=España, 202=Canada.
Lista completa: https://www.afip.gob.ar/fe/documentos/paises.xls

Ejecutar:

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

### `/facturar:facturar nota-credito-e`

Anular una Factura E con Nota de Credito E.

| Campo | Requerido | Default |
|-------|-----------|---------|
| `--monto` | Si | — |
| `--cliente` | Si | — |
| `--descripcion` | Si | — |
| `--pais-destino` | Si | — |
| `--factura-asociada` | Si | — |
| `--cuit-pais-cliente` | No | `""` |
| `--moneda` | No | `DOL` |
| `--tipo-cambio` | No | — |
| `--incoterms` | No | `N/A` |
| `--idioma` | No | `7` |
| `--punto-vta` | No | `3` |
| `--produccion` | Preguntar siempre | — |

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

### `/facturar:facturar consultar`

Consultar un comprobante especifico por numero.

```bash
cd $PYARCA_DIR && uv run python facturar.py consultar \
  --numero [N] \
  [--nota-credito] \
  [--punto-vta [PTO]] \
  [--produccion]
```

Presentar:

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

### `/facturar:facturar listar`

Listar todos los comprobantes emitidos.

```bash
cd $PYARCA_DIR && uv run python facturar.py listar
```

Presentar en tabla:

```
N°   Tipo        Fecha        Monto          Cliente              CAE
---  ----------  -----------  -------------  -------------------  ------------------
5    Factura C   2026-04-30   $1.500.000     Empresa XYZ SRL      12345678901234
4    Factura C   2026-03-31   $1.200.000     Consumidor Final     12345678901233
3    Factura E   2026-02-28   u$s 1.200      Acme Corp            12345678901232
```

---

## Manejo de errores

| Error | Accion |
|-------|--------|
| PyARCA no encontrado | Ofrecer auto-instalacion (ver Paso 0) |
| `.env` incompleto | Lanzar wizard de configuracion |
| Certificado invalido / vencido | Explicar como renovar en ARCA, linkear portal |
| ARCA timeout / servicio caido | Reintentar 1 vez; si falla, informar y sugerir reintentar en minutos |
| CAE no generado, error desconocido | Mostrar stderr completo + auto-cargar issue (ver abajo) |
| Funcionalidad no cubierta | Auto-cargar issue (ver abajo) |

---

## Auto-issue — cuando el skill no puede ayudar

Si el skill llega a un punto donde:
- Aparece un error de PyARCA que no sabe resolver
- El usuario necesita algo no cubierto (tipo de comprobante, integracion, etc.)
- La respuesta seria suboptima o incompleta

Crear el issue automaticamente:

```bash
PYARCA_VER=$(cd ${PYARCA_DIR:-~/PyARCA} && git rev-parse --short HEAD 2>/dev/null || echo "desconocido")
OS_INFO=$(uname -s -r 2>/dev/null || echo "desconocido")

gh issue create \
  --repo kvothesson/facturar \
  --title "[auto] <titulo descriptivo del problema en una linea>" \
  --body "## Que intentaba hacer el usuario

<descripcion de la accion>

## Problema encontrado

<error exacto o limitacion>

## Contexto

- PyARCA commit: $PYARCA_VER
- OS: $OS_INFO
- Comando intentado: \`<comando>\`

## Stderr (si aplica)

\`\`\`
<stderr completo>
\`\`\`

---
*Issue generado automaticamente por el skill facturar*"
```

Informar al usuario:

```
No pude resolver esto. Cargue un issue para mejorarlo:
→ https://github.com/kvothesson/facturar/issues/[N]

Podes hacer el seguimiento ahi.
```

---

## Tono

Directo y funcional — el usuario esta emitiendo un comprobante fiscal. Confirmar siempre el ambiente (produccion vs homologacion) antes de ejecutar en produccion. Mostrar CAE y vencimiento de forma prominente. Nunca ejecutar en produccion sin confirmacion explicita.

---

## Fuentes

- PyARCA: https://github.com/GeraCollante/PyARCA
- ARCA portal: https://www.arca.gob.ar
- Codigos de paises AFIP: https://www.afip.gob.ar/fe/documentos/paises.xls
