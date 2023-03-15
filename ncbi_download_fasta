#!/bin/bash

#
# descarga archivos fasta desde el ncbi
#

# funcion que descarga archivos fasta del NCBI
ncbi_download(){

  # asignar especie a descargar
  especie="${OPTARG}"
  # nombre corto a nivel de genero
  genero=$(echo "${especie}" | cut -d " " -f "1")

  # descarga todos los genomas del NCBI, solo si no estan ya descargados
  if [[ ! -e assembly_summary_refseq.txt ]]; then
     rsync -tv rsync://ftp.ncbi.nlm.nih.gov/genomes/ASSEMBLY_REPORTS/assembly_summary_refseq.txt ./
  fi

  # filtra aquellos de la especie de interes y devuelve solo nivel de ensamble y ruta de ubicacion
  grep -E "${especie}" assembly_summary_refseq.txt | cut -f "12,20" > ftp_links_${genero}.txt

  # da el numero de genomas disponibles totales y por nivel de ensamble
  echo -e "NCBI tiene un total de:" "'"$(cat ftp_links_${genero}.txt | wc -l)"'" "genomas para la especie '${especie}'"
  echo $(cat ftp_links_${genero}.txt | grep -c "Contig") "genomas nivel: Contig (menor calidad)"
  echo $(cat ftp_links_${genero}.txt | grep -c "Scaffold") "genomas nivel: Scaffold"
  echo $(cat ftp_links_${genero}.txt | grep -c "Chromosome") "genomas nivel: Chromosome"
  echo $(cat ftp_links_${genero}.txt | grep -c "Complete") "genomas nivel: Complete (mayor calidad)"

  # recibir input del usuario numero y nivel de ensambles deseados
  echo -e "\nPor favor elige el nivel de completud de ensamble que deseas"
  read -p "Elige entre: (Contig, Scaffold, Chromosome o Complete): " cal
  echo -e "\nPor favor introduce el numero de genomas que deseas descargar"
  read -p "Numero de Genomas: " genum

  # crear otra funcion para descargar todos los archivos del NCBI
  awk 'BEGIN{FS=OFS="/";filesuffix="genomic.fna.gz"}{ftpdir=$0;asm=$10;file=asm"_"filesuffix;print "rsync -t -v "ftpdir,file" ./"}' \
  ftp_links_${genero}.txt | sed 's/https/rsync/g' | grep ${cal} | sed 's/Contig//g;s/Scaffold//g;s/Chromosome//g;s/Complete Genome//g' \
  | head -${genum} > download_fna_files_${genero}.sh

  # descargar los genomas
  bash download_fna_files_${genero}.sh

  # crear carpeta y mover genomas descargados al nuevo directorio
  mkdir ASSEMBLY_${genero}
  mv GC*.fna.gz ASSEMBLY_${genero}

  # eliminafr archivos temporales
  rm ftp_links_Pseudomonas.txt download_fna_files_${genero}.sh
}

# funcion para pbtener estadisticas
estadisticas(){

  # crear directorio de resultados
  mkdir ASSEMBLY_${genero}/Stats

  # volver variable directorio de resultados
  dir="ASSEMBLY_${genero}/Stats"

  # --------------------------------------------------------------------
  # primer loop, creacion de archivos de estadisticas con assembly-stats
  # --------------------------------------------------------------------
  # NOTA: assembly-stats no obtiene profundidad, se debe obtener por separado

  for ensamble in ASSEMBLY_${genero}/*.gz; do
     ename=$(basename ${ensamble} | cut -d "_" -f "1,2") # asignar nombre de ensamble ( más corto)
     assembly-stats ${ensamble} > ${dir}/${ename}-stats.txt # crea un archivo de stats por ensamble
  done

  # ------------------------------------------------------------------------------------
  #    segundo loop, creacion del archivo final formato .csv con todas las stadisticas
  # ------------------------------------------------------------------------------------

  # Generar un archivo y poner los nombres de columnas
  echo -e "ID,Contigs,Length,Largest_contig,N50,N90,N_count,Gaps" > ${dir}/estadisticas_ensamble_${genero}.csv

  # ordenar y agrupar estadisticas a partir de estadisticas obtenidas por 1er loop
  for file in ${dir}/*stats*; do
     # asignar nombre de ensamble (corto)
     fname="$(basename $file | cut -d '-' -f '1')"
     # obtener numero de contigs
     contigs=$(cat ${file} | sed -n '2p' | cut -d ',' -f '2' | cut -d ' ' -f '4')
     # obtener longitud del ensamble
     length=$(cat ${file} | sed -n '2p' | cut -d ',' -f '1' | cut -d ' ' -f '3')
     # obtener tamaño de contig mas grande
     largest=$(cat ${file} | sed -n '2p' | cut -d ',' -f '4' | cut -d ' ' -f '4')
     # obtener N50
     N50=$(cat ${file} | sed -n '3p' | cut -d ',' -f '1' | cut -d ' ' -f '3')
     # obtener N90
     N90=$(cat ${file} | sed -n '7p' | cut -d ',' -f '1' | cut -d ' ' -f '3')
     # obtener Ns
     n_count=$(cat ${file} | sed -n '9p' | cut -d ',' -f '1' | cut -d ' ' -f '3')
     # obtener gaps
     gaps=$(cat ${file} | sed -n '10p' | cut -d ',' -f '1' | cut -d ' ' -f '3')
     # crea filas de archivo con datos obtenidos en las 3 filas anteriores
     echo -e "$fname,$contigs,$length,$largest,$N50,$N90,$n_count,$gaps"
  # anexa lo generado por el loop en el archivo creado antes del loop
  done >> ${dir}/estadisticas_ensamble_${genero}.csv

  # eliminar '-stats.txt'
  sed -i 's/-stats.txt//g' ${dir}/estadisticas_ensamble_${genero}.csv

  # convertir archivo CSV a TSV
  sed 's/,/\t/g' ${dir}/estadisticas_ensamble_${genero}.csv > ${dir}/estadisticas_ensamble_${genero}.tsv

}

# menu de ayuda
AYUDA="
   USO: $(basename ${0}) -e 'Pseudomonas aeruginosa'

   h)\tMuestra este menu de ayuda

   e)\tEspecie e interes
"

# si no hay argumento manda mensaje de ayuda y sal
if [[ ${#} -eq 0 ]]; then
   echo -e "${AYUDA}"
   exit 1
fi

# parseo de argumentos
while getopts e:h opt; do
   case "${opt}" in
      h)
        echo -e "${AYUDA}"
        ;;
      e)
        ncbi_download
        estadisticas
        ;;
      *)
        echo "La funcion ${0}, espera al menos un argumento 'especie'"
        echo -e "${AYUDA}"
        ;;
   esac
done
