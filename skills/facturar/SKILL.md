---
description: Facturacion electronica para Monotributo argentino via PyARCA. Emite Facturas C, NC C, Facturas E y NC E contra ARCA. Usalo con /facturar:facturar factura, nota-credito, factura-e, nota-credito-e, consultar, listar.
---

# Skill: /facturar:facturar

Wrapper de lenguaje natural sobre [PyARCA](https://github.com/GeraCollante/PyARCA). Traduce lo que el usuario describe a comandos PyARCA y los ejecuta via Bash.

## Prerequisito

PyARCA instalado y `.env` configurado con certificado digital de ARCA. Si no esta listo, mostrar:

```
Para usar este skill necesitas PyARCA configurado:
1. git clone https://github.com/GeraCollante/PyARCA && cd PyARCA
2. uv sync
3. Copiar .env.example a .env y completar con tu CUIT, certificado y clave privada de ARCA
Ver README de PyARCA para obtener el certificado digital.
```

Detectar si PyARCA esta disponible antes de ejecutar:
```bash
ls $PYARCA_DIR/facturar.py 2>/dev/null || echo "PyARCA no encontrado"
```

Si `PYARCA_DIR` no esta definido, buscar en paths comunes: `~/PyARCA`, `./PyARCA`, `/opt/PyARCA`.

---

## Comandos

### `/facturar:facturar factura`

Emitir una Factura C (consumidor final o empresa, mercado interno).

Preguntar al usuario si no estan claros:
- **Monto** en pesos (obligatorio)
- **Cliente** (default: "Consumidor Final")
- **Descripcion del servicio** (obligatorio)
- **Periodo**: desde / hasta en formato AAAAMMDD (obligatorio)
- **CUIT del cliente** (opcional, default: 0 = Consumidor Final)
- **Punto de venta** (default: 3)
- **Ambiente**: produccion o homologacion (preguntar siempre)

Construir y ejecutar:
```bash
cd $PYARCA_DIR && uv run python facturar.py factura \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --descripcion "[DESCRIPCION]" \
  --desde [DESDE] \
  --hasta [HASTA] \
  [--cuit-cliente [CUIT]] \
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

Anular o ajustar una Factura C con una Nota de Credito C.

Preguntar:
- **Monto** a anular
- **Numero de la factura asociada** (obligatorio)
- **Cliente** (debe coincidir con la factura original)
- **Ambiente**

Construir y ejecutar:
```bash
cd $PYARCA_DIR && uv run python facturar.py nota-credito \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --factura-asociada [NUMERO] \
  [--produccion]
```

Mostrar resultado igual que factura, indicando "Nota de Credito C".

---

### `/facturar:facturar factura-e`

Emitir una Factura E (exportacion de servicios al exterior).

Preguntar ademas de los campos basicos:
- **CUIT del pais cliente** (ej: 50000000016 para USA)
- **Codigo de pais destino** (ej: 200 = USA, 101 = Alemania)
- **Moneda** (DOL para dolares, PES para pesos)
- **Tipo de cambio** (cotizacion del dia)
- **Idioma del comprobante** (7=Español, 1=Ingles, 2=Portugues)

Sugerir fetchear el tipo de cambio oficial del BCRA si el usuario no lo sabe:
```bash
curl -s "https://dolarapi.com/v1/dolares/oficial" | jq '.venta'
```

Construir y ejecutar:
```bash
cd $PYARCA_DIR && uv run python facturar.py factura-e \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --descripcion "[DESCRIPCION]" \
  --desde [DESDE] \
  --hasta [HASTA] \
  --cuit-pais-cliente [CUIT_PAIS] \
  --pais-destino [COD_PAIS] \
  --moneda [MONEDA] \
  --tipo-cambio [TC] \
  --idioma [IDIOMA] \
  [--produccion]
```

---

### `/facturar:facturar nota-credito-e`

Anular una Factura E con Nota de Credito E.

Mismo flujo que `nota-credito` mas los campos de exportacion (pais, moneda, tipo de cambio).

```bash
cd $PYARCA_DIR && uv run python facturar.py nota-credito-e \
  --monto [MONTO] \
  --cliente "[CLIENTE]" \
  --factura-asociada [NUMERO] \
  --cuit-pais-cliente [CUIT_PAIS] \
  --pais-destino [COD_PAIS] \
  --moneda [MONEDA] \
  --tipo-cambio [TC] \
  [--produccion]
```

---

### `/facturar:facturar consultar`

Ver el ultimo comprobante emitido o consultar uno especifico.

```bash
cd $PYARCA_DIR && uv run python facturar.py consultar [--numero N] [--produccion]
```

Mostrar:
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

Listar comprobantes emitidos.

```bash
cd $PYARCA_DIR && uv run python facturar.py listar [--produccion]
```

Presentar en tabla:
```
N°   Tipo        Fecha        Monto        Cliente              CAE
---  ----------  -----------  -----------  -------------------  -------------------
5    Factura C   2026-04-30   $1.500.000   Empresa XYZ SRL      12345678901234
4    Factura C   2026-03-31   $1.200.000   Consumidor Final     12345678901233
...
```

---

## Deteccion de PYARCA_DIR

En orden de prioridad:
1. Variable de entorno `$PYARCA_DIR`
2. `~/PyARCA`
3. `./PyARCA`
4. `/opt/PyARCA`

Si no se encuentra en ninguno, instruir al usuario para clonar PyARCA y configurar `PYARCA_DIR`.

---

## Manejo de errores

| Error | Respuesta |
|-------|-----------|
| `facturar.py` no encontrado | Mostrar instrucciones de instalacion de PyARCA |
| `.env` incompleto / certificado invalido | Indicar que campo falta y linkear guia de PyARCA |
| Error ARCA (timeout, servicio caido) | Reintentar en modo homologacion para verificar el comando, luego reintentar produccion |
| CAE no generado | Mostrar el error exacto de ARCA para diagnostico |

---

## Tono

Directo y funcional. El usuario esta emitiendo un comprobante fiscal — claridad ante todo. Confirmar siempre el ambiente (produccion vs homologacion) antes de ejecutar. Mostrar el CAE y vencimiento de forma prominente — son los datos criticos.

---

## Fuentes

- PyARCA: https://github.com/GeraCollante/PyARCA
- ARCA (ex-AFIP) portal: https://www.arca.gob.ar
- Codigos de paises AFIP: https://www.afip.gob.ar/fe/documentos/paises.xls
