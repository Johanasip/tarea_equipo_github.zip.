# tarea_equipo_github.zip.
import os
import zipfile

# Crear estructura de carpetas y archivos
base_dir = "/mnt/data/tarea_equipo_github"
os.makedirs(base_dir, exist_ok=True)

branches = {
    "branch-1-posesion-vs-linea": """# Variables: 1) Mención de la Posesión Demoníaca Central  —  Línea Clave

## Variables incluidas
- **Posesión_Demoníaca_Central** (Categoría): niveles categorizados — Alto / Medio / Bajo.
- **Linea_Clave_Mensaje** (Texto representativo): ejemplo de frase clave: "You gave me your heart, now I'm here for your soul" y sus traducciones.

## Relación entre las variables
Ambas variables describen la **misma idea temática** desde dos enfoques: la primera mide la intensidad temática (cuánto se alude a posesión/maldad) y la segunda registra la frase concreta que resume esa intención. Son complementarias: la frase aporta evidencia cualitativa para la clasificación cuantitativa.

## Similitudes
- Ambas se refieren al **concepto de control / alma / posesión**.
- Se usan para identificar el tono oscuro del texto y la intención del narrador.

## Qué miden
- **Posesión_Demoníaca_Central**: mide la intensidad/visibilidad del motivo de posesión en la letra (escala ordinal: Bajo, Medio, Alto).
- **Linea_Clave_Mensaje**: registra el extracto textual que ejemplifica esa intención; se puede convertir a indicadores (p. ej. presencia de palabras clave: "soul", "alma", "possession").

## Por qué son medibles y comparables
- La variable ordinal (Bajo/Medio/Alto) se obtiene mediante reglas claras (p. ej. "Alto" si aparecen >2 alusiones directas o metáforas persistentes).
- La frase clave se puede tokenizar y buscar la presencia de tokens/palabras clave. A partir de eso se generan frecuencias o indicadores binarios (0/1).
- Comparables porque ambas pueden convertirse a números: (Alto=3, Medio=2, Bajo=1) y frecuencia de tokens en la frase.

## Método de medición propuesto
1. Normalizar texto (minúsculas, remover puntuación).
2. Buscar tokens clave: ["soul", "alma", "possession", "possession-related synonyms"].
3. Definir regla ordinal:
   - Alto = 3 o más tokens relevantes o metáforas fuertes en >2 líneas.
   - Medio = 1-2 tokens/metáforas.
   - Bajo = 0 tokens directos; solo alusiones débiles.
4. Para la línea clave -> extraer y almacenar el string; luego calcular presencia de tokens y longitud.

## Ejemplo
- Texto original contiene la línea: "You gave me your heart, now I'm here for your soul" → `Posesión_Demoníaca_Central = Alto` ; `Linea_Clave_Mensaje = "You gave me your heart, now I'm here for your soul"`
""",
    "branch-2-pecado-frecuencia": """# Variables: 2) Uso de la palabra "Pecado" o sinónimos — Frecuencia

## Variables incluidas
- **Pecado_Frecuencia** (numérica): número de ocurrencias de "sinónimos de pecado" por versión.
- **Pecado_Directo** (binaria/ordinal): indicador de si se usa la palabra exacta "pecado" (0 = no, 1 = sí) o suavizaciones.

## Relación entre las variables
La frecuencia cuantifica cuántas alusiones a la transgresión hay; el indicador binario/ordinal muestra si se usa la palabra exacta o se suaviza. Juntas permiten comparar literalidad vs. eufemismo.

## Similitudes
- Ambas miden cómo se trata el concepto de transgresión.
- Se extraen del mismo texto y se pueden derivar por el mismo pipeline de tokenización.

## Qué miden
- **Pecado_Frecuencia**: cuenta de ocurrencias (0,1,2,...).
- **Pecado_Directo**: presencia exacta o su ausencia (0/1) o grado (0 = none, 1 = eufemismo, 2 = palabra directa).

## Por qué son medibles y comparables
- La frecuencia es un conteo objetivo.
- La presencia directa se define por reglas de string-matching (exact match) o por lista de sinónimos aprobada.
- Comparables entre versiones (original, español, filipino) porque todas usan texto alineable para conteo.

## Método de medición propuesto
1. Lista de términos: ["sin", "sin(s)", "pecado", "sinful", "transgression", "error", ...].
2. Tokenización y conteo por archivo/version.
3. Clasificación de `Pecado_Directo`:
   - 2 = palabra "pecado" exacta encontrada.
   - 1 = sinónimos/eufemismos encontrados.
   - 0 = ninguno.

## Ejemplo
- Versión Inglés: "I'm the only one who'll love your sins" → `Pecado_Frecuencia = 1`, `Pecado_Directo = 1 (sin/sins)`.
""",
    "branch-3-coreano-presencia": """# Variables: 3) Presencia de vocabulario en Coreano — Conteo de palabras no traducidas

## Variables incluidas
- **Coreano_Presencia_Count** (numérica): número de palabras/frases en coreano que se mantienen sin traducir.
- **Coreano_Porcentaje_Lineas** (porcentaje): proporción de líneas que contienen al menos una palabra coreana.

## Relación entre las variables
Ambas miden la misma característica desde distinta métrica: conteo absoluto y proporción relativa. Permiten comparar literalidad cultural entre versiones.

## Similitudes
- Extraídas mediante detección de caracteres Hangul o comparación con lista de tokens coreanos.
- Útiles para medir fidelidad cultural (si se mantienen términos originales).

## Qué miden
- **Coreano_Presencia_Count**: conteo absoluto de tokens Hangul.
- **Coreano_Porcentaje_Lineas**: número de líneas con Hangul dividido por total de líneas.
""",
    "branch-4-tematica-enfasis": """# Variables: 4) Énfasis Temático Dominante — Descripción temática

## Variables incluidas
- **Tematica_Principal** (categoría): e.g., "Seducción y Posesión Fría", "Dominio y Talismán", "Vínculo Obscuro/Éxtasis".
- **Intensity_Score** (numérica): puntuación de 0-10 que cuantifica cuánto domina ese énfasis en la versión.
""",
    "branch-5-registro-vocal": """# Variables: 5) Categoría de Registro Vocal — Percepción del tono

## Variables incluidas
- **Registro_Vocal_Tono** (categoría): e.g., "Grave y Agresivo", "Medio-Agudo y Limpio", "Grave y Fiel a la Energía".
- **Registro_Similarity_Score** (numérica 0-1): medida de cuán cercano es el registro a la versión original (si existe audio de referencia).
"""
}

# Crear carpetas y archivos para cada rama
for branch, content in branches.items():
    branch_dir = os.path.join(base_dir, branch)
    os.makedirs(branch_dir, exist_ok=True)
    with open(os.path.join(branch_dir, "variables.md"), "w", encoding="utf-8") as f:
        f.write(content)
    with open(os.path.join(branch_dir, "historial.md"), "w", encoding="utf-8") as f:
        f.write(f"# Historial de pasos - {branch}\n\n1. Extracción y definición de variables.\n2. Redacción de documentación.\n3. Validación grupal.\n4. Archivo listo para subir a GitHub.\n")

# Crear archivo resumen_equipo.md
resumen = """# Resumen del trabajo del equipo

## Objetivo
Documentar variables de la base de datos de Huntrix y crear un repositorio en GitHub con 5 ramas, cada una con dos variables relacionadas.

## Estructura
- branch-1-posesion-vs-linea
- branch-2-pecado-frecuencia
- branch-3-coreano-presencia
- branch-4-tematica-enfasis
- branch-5-registro-vocal

## Contenido
Cada rama contiene:
- `variables.md`: descripción de las variables.
- `historial.md`: pasos realizados.

## Equipo
- Integrante 1
- Integrante 2
- Integrante 3
- Integrante 4

## Fecha límite
Viernes 24 de octubre, 11:52 a.m.
"""

with open(os.path.join(base_dir, "resumen_equipo.md"), "w", encoding="utf-8") as f:
    f.write(resumen)

# Crear ZIP
zip_path = "/mnt/data/tarea_equipo_github.zip"
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as zipf:
    for root, dirs, files in os.walk(base_dir):
        for file in files:
            zipf.write(os.path.join(root, file),
                       os.path.relpath(os.path.join(root, file), base_dir))

zip_path
