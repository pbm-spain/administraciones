# Administraciones España - Directorio de recursos públicos

Base de datos en JSON con recursos de la administración pública española en sus tres niveles: estatal, autonómica y municipal. Para cada recurso se mantiene URL principal, sede electrónica (cuando aplica), organismo, descripción, hashtags y servicios principales.

## Estructura del repositorio

```
.
├── admin_estatal.json      # Administración General del Estado (ministerios, AEAT, SS, etc.)
├── admin_ccaa.json         # Comunidades y Ciudades Autónomas (17 + 2)
├── admin_municipal.json    # Ayuntamientos, diputaciones y entes locales
└── CLAUDE.md
```

Cada archivo es un array JSON. Una entrada típica tiene esta forma:

```json
{
  "id": "aeat",
  "nombre": "Agencia Tributaria",
  "url": "https://www.agenciatributaria.es",
  "sede_electronica": "https://sede.agenciatributaria.gob.es",
  "organismo": "Ministerio de Hacienda",
  "nivel": "estatal",
  "tipo": "agencia",
  "descripcion": "Gestión del sistema tributario estatal y aduanero",
  "hashtags": ["#AEAT", "#Hacienda", "#Impuestos"],
  "servicios_principales": ["Renta", "IVA", "Sociedades", "Aduanas"]
}
```

### Campos por archivo

| Campo                    | estatal | ccaa | municipal | Notas |
|--------------------------|:-------:|:----:|:---------:|-------|
| `id`                     | sí      | —    | —         | slug único |
| `nombre`                 | sí      | sí   | sí        | nombre oficial |
| `url`                    | sí      | sí   | sí        | portal principal |
| `sede_electronica`       | sí      | —    | —         | URL de la sede-e si difiere |
| `organismo`              | sí      | sí   | —         | ministerio o consejería matriz |
| `nivel`                  | sí      | sí   | sí        | `estatal` / `autonomico` / `municipal` |
| `tipo`                   | sí      | sí   | sí        | ministerio, agencia, consejería, ayuntamiento… |
| `descripcion`            | sí      | sí   | sí        | una línea con el ámbito competencial |
| `hashtags`               | sí      | sí   | sí        | array de etiquetas con `#` |
| `servicios_principales`  | sí      | sí   | sí        | array de trámites o áreas clave |
| `comunidad_autonoma`     | —       | sí   | sí        | en municipios indica la CCAA donde se ubica |
| `municipio`              | —       | —    | sí        | nombre del municipio |
| `provincia`              | —       | —    | sí        | provincia administrativa |

## Cómo mantener los datos

- **Verificación de URLs**: comprobar periódicamente que todas las URLs respondan con 200 y que la sede electrónica siga apuntando al endpoint vigente (las sedes cambian de dominio con frecuencia, por ejemplo `*.gob.es` → `sede.*.gob.es`).
- **Altas**: añadir nuevos recursos al array correspondiente al final del archivo. Preservar el formato JSON canónico (UTF-8, dos espacios de indentación, sin comas finales).
- **Hashtags**: mantenerlos consistentes entre archivos. Reutilizar etiquetas ya existentes antes de inventar nuevas (`#AEAT` y no `#AgenciaTributaria` si la primera ya está en uso). Empezar siempre con `#` y usar PascalCase para varias palabras.
- **`servicios_principales`**: lista corta (3-6 elementos) con los trámites o áreas más buscadas por el ciudadano, no un listado exhaustivo.
- **Niveles**: una entrada vive en exactamente un archivo. Los entes que actúan en varios niveles (por ejemplo, una entidad pública estatal con delegaciones autonómicas) van en `admin_estatal.json`.
- **Identificadores**: en `admin_estatal.json` el `id` es un slug en minúsculas, sin espacios ni acentos (`aeat`, `seg-social`, `sepe`). Si añades un campo `id` a CCAA o municipios, sigue la misma convención.

## Reglas de estilo

- JSON válido y formateado (`python3 -m json.tool` o equivalente como verificación).
- Sin claves duplicadas dentro de una misma entrada.
- URLs con esquema `https://` siempre que el organismo lo soporte.
- Descripciones en español, neutras, sin valoraciones.
- No introducir campos nuevos sin actualizar también esta tabla.

## Verificaciones rápidas

```bash
# JSON válido
for f in admin_*.json; do python3 -m json.tool "$f" > /dev/null && echo "$f OK"; done

# Recuento de entradas por archivo
for f in admin_*.json; do echo "$f: $(python3 -c "import json,sys; print(len(json.load(open('$f'))))")"; done

# URLs duplicadas
python3 -c "
import json, collections
for f in ['admin_estatal.json','admin_ccaa.json','admin_municipal.json']:
    urls = [e['url'] for e in json.load(open(f))]
    dups = [u for u,c in collections.Counter(urls).items() if c>1]
    print(f, 'duplicates:', dups or 'none')
"
```
