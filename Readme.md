# Reconstrucción de Transcritos con StringTie

## Introducción

StringTie es un programa de código abierto utilizado para ensamblar y cuantificar transcritos a partir de datos de secuenciación de ARN (RNA-seq). Esta guía proporciona una descripción detallada de los parámetros utilizados en StringTie y cómo ajustarlos para personalizar el proceso de ensamblaje y cuantificación.

## Parámetros de StringTie

### Parámetros Básicos

- **-x**: Especifica el nombre del archivo de salida (archivo GTF) que contendrá la información de los transcritos ensamblados.
- **-i**: Opcional. Proporciona un archivo de referencia de anotación de genes en formato GTF o GFF3 para guiar el proceso de ensamblaje y cuantificación.
- **-u**: Especifica el nombre del archivo de salida que contendrá la matriz de expresión de los genes y sus isoformas.
- **-G**: Utiliza la anotación proporcionada con el parámetro -i para guiar el proceso de ensamblaje y cuantificación.

### Modo de Fusión de Transcritos (`--merge`)

Este modo combina y ensambla conjuntos de transcritos provenientes de múltiples archivos GTF/GFF en un conjunto no redundante de transcritos. Es esencial para el análisis diferencial de expresión génica.

- **-G <guide_gff>**: Especifica un archivo de anotación de referencia para incluir en la fusión.
- **-o <out_gtf>**: Establece el nombre del archivo de salida para los transcritos fusionados en formato GTF.
- **-m <min_len>**: Especifica la longitud mínima que debe tener un transcrito de entrada para ser incluido en la fusión (por defecto, 50 bases).
- **-c <min_cov>**: Establece la cobertura mínima que debe tener un transcrito de entrada para ser incluido en la fusión (por defecto, 0).
- **-F <min_fpkm>**: Define el FPKM mínimo que debe tener un transcrito de entrada para ser incluido en la fusión (por defecto, 0).
- **-T <min_tpm>**: Establece el TPM mínimo que debe tener un transcrito de entrada para ser incluido en la fusión (por defecto, 0).
- **-f <min_iso>**: Especifica la fracción mínima de isoformas que un transcrito debe representar para ser incluido en la fusión (por defecto, 0.01).
- **-i**: Conserva los transcritos fusionados que contengan intrones retenidos.
- **-l <label>**: Prefijo de nombre para los transcritos de salida fusionados (por defecto, MSTRG).

## Ejemplo de Uso de StringTie

A continuación, se presenta un ejemplo de cómo utilizar StringTie para ensamblar y cuantificar transcritos a partir de datos de RNA-seq:

```bash
stringtie <read_alignments.bam> -o output.gtf -p 10 -f 0.05 -m 200 --conservative -t -s 3.0 -l LNCRNA_
```

Donde:
- `<read_alignments.bam>`: Especifica el archivo BAM de alineamientos como entrada.
- `-o output.gtf`: Establece el nombre del archivo de salida donde se escribirán los transcritos ensamblados.
- `-f 0.05`: Establece la mínima abundancia de isoformas como fracción de la isoforma más abundante en un locus.
- `-m 200`: Define la longitud mínima permitida para los transcritos predichos.
- `--conservative`: Activa el modo conservador para ensamblar transcritos de manera más estricta.
- `-t`: Desactiva el recorte de los extremos de los transcritos.
- `-s 3.0`: Establece la mínima cobertura permitida para los transcritos de una sola exon.
- `-l LNCRNA_`: Personaliza el prefijo para el nombre de los transcritos de salida.

## Consideraciones Adicionales

- **-f 0.05**: Permite la identificación de isoformas menos abundantes.
- **-m 200**: Permite la identificación de transcritos más cortos.
- **-s 3.0**: Ayuda a filtrar transcritos con una cobertura más baja.
- **--conservative**: Reduce la tasa de falsos positivos.

## Modo de Fusión de Transcritos

Para el modo de fusión de transcritos, se puede utilizar el siguiente comando:

```bash
stringtie --merge -o NAPL5_0.05.gtf -p 10 -m 200 -f 0.05 -T 0.5 -l LNCRNA_ SRR14404330.gtf SRR14404331.gtf SRR14404332.gtf SRR14404333.gtf SRR14404334.gtf SRR14404335.gtf
```

En este caso, se agrega el parámetro `-T 0.5` para incluir los transcritos con baja expresión, mejorando los procesos posteriores de normalización.

## Anotación de Transcritos con gffcompare

Para mejorar la precisión y la integridad de la anotación de los transcritos ensamblados, se utilizó **gffcompare**. Este programa compara y anota los transcritos ensamblados contra un archivo de referencia de anotación de genes. A continuación, se describe el proceso y el comando utilizado:

### Proceso de Anotación

El archivo de referencia utilizado fue el archivo GTF de *Rattus norvegicus* (genomic.gff). El objetivo de esta comparación es identificar y clasificar los transcritos ensamblados en función de su concordancia con los genes anotados en la referencia. Este paso es crucial para validar la calidad del ensamblaje y para identificar nuevas isoformas potenciales.

### Comando Utilizado

El siguiente comando se utilizó para llevar a cabo la anotación con gffcompare:

```bash
gffcompare -r genomic.gff -o rn7 merged_transcripts.gtf
```

Donde:

- **-r genomic.gff**: Especifica el archivo de referencia de anotación de genes en formato GFF.
- **-o rn7**: Establece el prefijo para los archivos de salida generados por gffcompare.
- **merged_transcripts.gtf**: Especifica el archivo GTF resultante del proceso de fusión de transcritos.

### Resultados y Consideraciones

El uso de gffcompare permitió:

1. **Validación de Transcritos**: Confirmar la presencia de transcritos conocidos y la identificación de nuevas isoformas.
2. **Clasificación de Transcritos**: Clasificar los transcritos ensamblados en categorías como "completamente coincidente", "parcialmente coincidente" y "nuevas isoformas".
3. **Mejora de la Anotación**: Proporcionar una anotación más precisa y detallada de los transcritos ensamblados, lo cual es esencial para estudios posteriores de expresión génica y análisis funcional.

Este paso es fundamental para asegurar la calidad y la utilidad de los datos de RNA-seq en investigaciones genómicas y transcriptómicas.

## Filtrado de Transcritos con Script Personalizado

Antes de utilizar RNAmining, se empleó un script personalizado para filtrar los transcritos anotados por gffcompare. Este script separa los transcritos y sus exones según el código de clasificación de transcrito.

### Script Utilizado

El siguiente script en Python fue utilizado para filtrar los transcritos con los códigos de clasificación `=, u, i, x`:

```python
###################################################################################
#                                                                                 #
#                             Allan Peñaloza                                      #
#     un script para filtrar un archivo gtf anotado por Gffcompare                #
# y separa los transcritos y sus exones por codigo de clasificacion de transcrito #
#                                                                                 #
###################################################################################

import argparse

def procesar_archivo(archivo_entrada, archivo_salida, codigos):
    codigos = ['class_code "' + codigo + '"' for codigo in codigos.split(',')]
    with open(archivo_entrada, "r") as archivo_original:
        with open(archivo_salida, "w") as nuevo_archivo:
            anterior = None
            for linea in archivo_original:
                elemento = linea.strip().split("\t")
                campo = elemento[8]
                subcampos = campo.split("; ")
                ID = subcampos[1].split(" ")[1].replace('"', '')  # Extrae el ID correctamente
                if elemento[2] == 'transcript' and any(codigo in linea for codigo in codigos):
                    nuevo_archivo.write(linea)  # No añadimos un salto de línea
                    anterior = ID
                elif elemento[2] == 'exon' and anterior == ID:
                    nuevo_archivo.write(linea)  # No añadimos un salto de línea

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="""
                                     Negro Envidioso (allan.penaloza@ug.uchile.cl)
                                     
                                     Presenta:
                                     
                                     Script para filtrar un GTF reconstruido por StringTie por códigos de clasificación de transcripciones
                                     """, formatter_class=argparse.RawTextHelpFormatter)
    parser.add_argument('-i', '--input', help='El archivo GTF de entrada', required=True)
    parser.add_argument('-o', '--output', help='El archivo GTF de salida', required=True)
    parser.add_argument('-f', '--filter', nargs='?', help='Códigos de clasificación de transcripciones para filtrar. Ejemplo: -f o,c,k,m, etc.', required=False, default="")
    args = parser.parse_args()
    procesar_archivo(args.input, args.output, args.filter)
```

### Resultados del Filtrado

El script permitió filtrar los transcritos con los códigos de clasificación `=, u, i, x`. Los transcritos filtrados con los códigos `u, i, x` fueron posteriormente ingresados a RNAmining para evaluar su potencial no codificante.

## Análisis Posterior con RNAmining

Para el análisis posterior de los transcritos anotados y filtrados, se utilizó la herramienta web **RNAmining**. Esta herramienta proporciona una plataforma integrada para el análisis de datos de RNA-seq, facilitando la normalización, visualización y análisis diferencial de expresión génica.

### Proceso de Análisis

RNAmining permite cargar los archivos GTF anotados y realizar una serie de análisis bioinformáticos avanzados. Entre las funcionalidades destacadas se incluyen:

- **Normalización de Datos**: Ajuste de los datos de expresión para eliminar sesgos técnicos y biológicos.
- **Análisis Diferencial de Expresión**: Identificación de genes diferencialmente expresados entre condiciones experimentales.
- **Visualización de Datos**: Generación de gráficos y tablas para la interpretación de los resultados.

### Comando Utilizado

El análisis se realizó a través de la interfaz web de RNAmining, disponible en RNAmining.

### Resultados y Consideraciones

El uso de RNAmining permitió:

1. **Normalización Eficiente**: Ajustar los datos de expresión para obtener resultados más precisos y comparables.
2. **Identificación de Genes Clave**: Detectar genes con cambios significativos en su expresión entre diferentes condiciones.
3. **Visualización Clara**: Facilitar la interpretación de los resultados mediante gráficos y tablas intuitivas.

Este análisis es crucial para comprender los mecanismos biológicos subyacentes y para avanzar en la investigación genómica y transcriptómica.
