# facturar

Plugin de Claude Code — facturacion electronica para Monotributo argentino.

Wrapper de lenguaje natural sobre [PyARCA](https://github.com/GeraCollante/PyARCA). Le decis a Claude "emitir factura por $1.500.000 a Empresa XYZ por desarrollo de software abril 2026" y el arma y ejecuta el comando correcto contra ARCA.

## Prerequisito

PyARCA instalado y configurado con tu certificado digital de ARCA (se hace una sola vez).

```bash
git clone https://github.com/GeraCollante/PyARCA
cd PyARCA
uv sync
cp .env.example .env
# completar .env con CUIT, certificado y clave privada
```

Guia completa en el [README de PyARCA](https://github.com/GeraCollante/PyARCA).

## Instalacion

```bash
claude --plugin-dir /ruta/a/facturar
```

O instalar desde el marketplace de ar-plugins.

## Comandos y ejemplos

### `/facturar:facturar factura`

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

### `/facturar:facturar nota-credito`

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

### `/facturar:facturar factura-e`

Emitir Factura E (exportacion de servicios al exterior).

```
✅ Factura E emitida

Numero:      3
CAE:         12345678901236
Vencimiento: 2026-05-17
Monto:       u$s 1.200 (TC: $1.410 → $1.692.000 ARS)
Cliente:     Acme Corp
Pais:        USA (200)
Periodo:     2026-04-01 → 2026-04-30

Ambiente: Produccion
```

---

### `/facturar:facturar consultar`

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

### `/facturar:facturar listar`

Listar comprobantes emitidos.

```
N°   Tipo        Fecha        Monto          Cliente              CAE
---  ----------  -----------  -------------  -------------------  ------------------
5    Factura C   2026-04-30   $1.500.000     Empresa XYZ SRL      12345678901234
4    Factura C   2026-03-31   $1.200.000     Consumidor Final     12345678901233
3    Factura E   2026-02-28   u$s 1.200      Acme Corp            12345678901232
```

## Fuentes

- PyARCA: https://github.com/GeraCollante/PyARCA
- ARCA portal: https://www.arca.gob.ar
- Discusion original: https://www.reddit.com/r/merval/comments/1t5ivpv/comment/okdmyvd/
