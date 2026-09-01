# rastro-finint

Módulo de investigación financiera del kit **rastro**. Cuatro herramientas para análisis de cuentas bancarias, tarjetas, criptoactivos y trazabilidad de movimientos, pensadas para investigación policial y ciberinteligencia.

**100% del lado del cliente. Sin backend, sin APIs externas, sin telemetría.** Todo el procesamiento ocurre en el navegador de quien lo usa; nada se envía a ningún servidor y nada se persiste entre sesiones.

** Enlace a la herramienta: https://s3gad3.github.io/rastro-finint 

---

## Por qué existe

Las tareas de análisis financiero básico (validar un IBAN, identificar una red de tarjeta, comprobar el formato de una dirección cripto, reconstruir un grafo de movimientos a partir de un extracto) suelen depender de herramientas online que exigen subir datos sensibles a terceros. `rastro-finint` es un único archivo HTML autocontenido que hace todo eso localmente, para poder usarse con datos de un caso real sin que salgan del equipo del investigador.

## Módulos

### 01 · Cuentas / IBAN
- Valida el dígito de control de un IBAN (algoritmo ISO 7064 mod 97) o de un CCC español de 20 dígitos.
- Identifica la entidad bancaria española a partir del código de 4 dígitos, con una **tabla editable** desde la propia interfaz.
- Convierte CCC ↔ IBAN.

### 02 · BIN / Tarjetas
- Valida el número con el **algoritmo de Luhn**.
- Detecta la red (Visa, Mastercard, Amex, Discover, Diners, JCB, UnionPay, Maestro) por rango IIN público.
- Permite **cargar un CSV propio de rangos BIN** (columnas `bin`, `entidad`, `pais`, `tipo`) para resolver emisor sin depender de ningún servicio externo.

### 03 · Cripto — BTC / ETH
- Verifica formato y checksum de direcciones Bitcoin (Base58Check con SHA-256 vía Web Crypto API, y Bech32/Bech32m incluyendo SegWit v0–v16 y Taproot) y Ethereum (formato 0x + 40 hex, checksum EIP-55).
- **No consulta saldo ni histórico en vivo** — eso exigiría un nodo o un explorador externo, y se ha evitado a propósito para no requerir red. En su lugar, permite **importar un export CSV/JSON de un explorador** (Etherscan, mempool.space, Blockchair...) para reconstruir el grafo de trazabilidad offline.
- Directorio local editable de direcciones conocidas (exchanges, sospechosos identificados) para etiquetar nodos en el grafo.
- Enlaces directos a exploradores públicos para consulta manual (se abren solo al pulsarlos).

### 04 · Normalizador + Grafo de movimientos
- Carga extractos bancarios en **CSV, XLSX, TXT o Norma 43 (AEB)**.
- Detecta el formato automáticamente; para CSV/XLSX ofrece un mapeo de columnas corregible a mano.
- Normaliza todo a un esquema común (fecha, cuenta, contraparte, concepto, importe, saldo).
- Redacta un **resumen del análisis en lenguaje natural**: periodo cubierto, entradas/salidas/neto, evolución del saldo, contraparte de mayor volumen, movimiento más alto.
- Señala **patrones que pueden merecer revisión** (heurísticas, no conclusiones): movimientos justo por debajo de 3.000 €, grupos de importes similares repetidos en pocos días con la misma contraparte, alto porcentaje de contrapartes sin identificar.
- Construye un **grafo dirigido** de la cuenta investigada (nodo central fijo, admite varias cuentas propias) frente a sus contrapartes:
  - Por defecto, **una flecha por movimiento** — si hay 10 transferencias a la misma cuenta, se ven 10 flechas en abanico, no una agregada. Puede activarse el modo agregado para extractos muy voluminosos.
  - Selector para incluir **todos los movimientos, solo entradas, solo salidas, o una selección manual** marcada en la tabla.
  - Nodos etiquetados con los **dígitos finales** de cada cuenta (más útil que el inicio, que suele compartir entidad/oficina).
  - Ranking de principales contrapartes por volumen.
- Exporta el CSV normalizado.

## Cómo usarlo

No requiere instalación ni build. Es un único archivo HTML.

1. Descarga `rastro-finint.html`.
2. Ábrelo con cualquier navegador moderno (Chrome, Edge, Firefox).
3. Listo — funciona incluso sin conexión a internet, salvo la primera carga de las librerías JS (ver [Dependencias](#dependencias)).

También puede publicarse con GitHub Pages para tenerlo accesible desde una URL propia, sin que eso implique enviar datos a ningún sitio: el procesamiento sigue siendo enteramente local en el navegador de quien lo visita.

## Privacidad y custodia de la evidencia

- Ningún dato introducido (cuentas, tarjetas, direcciones, movimientos importados) se guarda entre sesiones ni se envía a ningún servidor.
- Al recargar la página se pierde todo — es una decisión de diseño deliberada para no dejar rastro de datos de un caso en el propio equipo o en almacenamiento del navegador.
- No usa `localStorage`, `sessionStorage`, cookies ni ningún backend propio.

## Dependencias

Tres librerías JS de terceros, cargadas por CDN (necesitan conexión solo la primera vez que se abre la página, luego el navegador las cachea):

- [PapaParse](https://www.papaparse.com/) — parseo de CSV.
- [SheetJS / xlsx](https://sheetjs.com/) — lectura de Excel.
- [D3.js](https://d3js.org/) — el grafo de trazabilidad.

Si necesitas un funcionamiento 100% air-gapped, descarga estas tres librerías y sustituye las URLs `<script src>` del `<head>` por las rutas locales.

## Limitaciones conocidas

- **BIN/tarjetas**: no incluye una base BIN completa incrustada, porque no se puede garantizar su exactitud sin mantenerla activamente. Se apoya en rangos IIN públicos y en el CSV que cargue cada usuario.
- **Cripto**: no hay consulta de saldo/histórico en vivo por diseño (sin red). El checksum EIP-55 de Ethereum se valida solo cuando la dirección mezcla mayúsculas/minúsculas; no se incluye una implementación de Keccak-256 para mantener el archivo ligero.
- **Norma 43**: el parser interpreta los anchos de campo estándar de la especificación AEB, pero algunos bancos usan variantes ligeras. Revisa siempre los primeros resultados contra el extracto original antes de usarlos como prueba.
- **Identificación de entidad por IBAN**: la tabla de códigos de banco españoles es parcial y editable; para certeza total, contrastar con el registro del Banco de España.
- Las señales de "posible fraccionamiento" u otros patrones destacados son heurísticas orientativas, no conclusiones periciales.

## Estructura del repositorio

```
rastro-finint.html   # herramienta completa (HTML + CSS + JS), archivo único
README.md
```

## Contribuir

Pull requests bienvenidas, especialmente para:
- Ampliar la tabla de códigos de entidad española (o añadir tablas de otros países).
- Mejorar el parser de Norma 43 con variantes específicas de bancos.
- Añadir soporte a otros formatos de extracto normalizados (MT940, CAMT.053...).

## Licencia

Pendiente de decisión 

## Aviso

Herramienta pensada como apoyo al análisis, no como prueba pericial concluyente por sí sola. Los resultados de validación de checksums, detección de patrones y normalización de datos deben revisarse y contrastarse siempre con las fuentes originales.
