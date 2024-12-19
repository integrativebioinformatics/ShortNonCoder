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
