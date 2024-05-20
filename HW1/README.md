# HW 1
- Найти в Sequence Read Archive небольшой датасет (или скачать часть датасета) - ампликонное секвенирование микробиоты кишечника человека (illumina MiSeq), прогнать через QIIIME2, но вместо dada2 использовать deblur. Представленности посчитать до уровня видов.

## 1. Установить QIIME2
- Загрузите и установите Miniconda, если он еще не установлен
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
```
# Перезапустите терминал, затем создайте и активируйте новое окружение для QIIME 2
conda update conda
wget https://data.qiime2.org/distro/core/qiime2-2023.2-py38-linux-conda.yml
conda env create -n qiime2-2023.2 --file qiime2-2023.2-py38-linux-conda.yml
conda activate qiime2-2023.2
```
- Проверяю, что все работает, если все сделано правильно, эта команда должна вывести справочную информацию о QIIME 2.
```
qiime --help
```
## 2. Идентификация переменных
- Загружаем данные с SRA.NCBI (в поиске вводим: gut human microbiome, в фильтрах можно указать Illumina, single).

Данные с которыми будем работать:
- SRR28725319.fastq
- SRR28725320.fastq
- SRR28725322.fastq
- SRR28725323.fastq
- SRR28725324.fastq

- Создаем папки qiime_test, qiime_formatted и задаем переменные:
```
input_reads=/home/anastasia/microbiome/qiime_test
manifest=/home/anastasia/microbiome/qiime_test/manifest.tmp

output_qza=/home/anastasia/microbiome/qiime_formatted/seq.qza
classifier=/home/anastasia/microbiome/silva138_AB_V4_classifier.qza
```

- Через nano исправляем пути в файле manifest.tmp:
```
sample-id       absolute-filepath
SRR28725319     /home/anastasia/microbiome/qiime_test/SRR28725319.fastq
SRR28725320     /home/anastasia/microbiome/qiime_test/SRR28725320.fastq
SRR28725322     /home/anastasia/microbiome/qiime_test/SRR28725322.fastq
SRR28725323     /home/anastasia/microbiome/qiime_test/SRR28725323.fastq
SRR28725324     /home/anastasia/microbiome/qiime_test/SRR28725324.fastq
```
## 3. Создание seq.qza
```
qiime tools import --type 'SampleData[SequencesWithQuality]' --input-path $manifest --output-path $output_qza --input-format SingleEndFastqManifestPhred33V2
```
- Команда может занять несколько минут, но ее необходимо выполить, она используется для преобразования файлов входных данных в артефакты Qiime2, которые представляют собой сжатые папки. Этот процесс необходим, потому что Qiime2 требует, чтобы все данные были в виде артефактов, чтобы отслеживать и регистрировать данные в системе.
  
## 4. Обрезка и кластеризация с помощью DADA2 или DEBLUR
- Создаем папку denoise_out и переносим туда 3 файла из папки mid: feature_table.qza, rep_seqs.qza, stats.qza. Команды могут занять продолжительное врема.
```
# DADA2
qiime dada2 denoise-single --i-demultiplexed-seqs $output_qza --p-trunc-len 249 --p-trunc-q 10 --o-table "denoise_out/feature_table.qza" --o-representative-sequences "denoise_out/rep_seqs.qza" --o-denoising-stats "denoise_out/stats.qza"
```
```
# DEBLUR
qiime deblur denoise-16S --i-demultiplexed-seqs $output_qza --p-trim-length 249 --p-sample-stats --o-representative-sequences "./denoise_out/deblur-rep-seqs.qza" --o-table "./denoise_out/deblur-table.qza" --o-stats "./denoise_out/deblur-stats.qza"
```
- Затем мы можем получить файл .qzv и использовать веб-сайт (https://view.qiime2.org/) для его визуализации.
```
qiime feature-table summarize --i-table ./denoise_out/deblur-table.qza --o-visualization ./denoise_out/deblur-table.qzv
```
- Или мы можем получить файл .fasta репрезентативных последовательностей и файл .csv статистики:
```
# Variable identification
feature_table="/home/anastasia/microbiome/denoise_out/feature_table.qza"
rep_seqs_qza="/home/anastasia/microbiome/denoise_out/rep_seqs.qza"

# EXPORT REPRESENTATIVE SEQUENCES AS .FASTA
qiime tools export --input-path $rep_seqs_qza --output-path "denoise_out/"
rep_seqs="denoise_out/dna-sequences.fasta"

# EXPORT DENOISING STATS
qiime tools export --input-path "denoise_out/deblur-stats.qza" --output-path "denoise_out/"
deblur_stats="denoise_out/stats.csv"
```
Вот как выглядит файл статистики .csv:
| Sample ID	| Reads Raw	| Unique Reads Derep | Reads Derep |	Unique Reads Deblur |	Reads Deblur |	Unique Reads Hit Artifact |	Reads Hit Artifact	| Unique Reads Chimeric |	Reads Chimeric |	Unique Reads Hit Reference |	Reads Hit Reference |	Unique Reads | Missed Reference |	Reads Missed Reference |
| -------- | ------- |-------- | ------- |-------- | ------- |-------- | ------- |-------- | ------- |-------- | ------- |-------- | ------- |
| SRR28725319 | 59458 | 3279 | 38380 | 166 | 21818 | 16 | 32 | 62| 137 | 49 | 21552 | 0 | 0 |
| SRR28725320 | 77484 | 3946 | 52165 | 210 | 30711 | 39 | 80 | 71 | 217 | 60 | 30326 | 0 | 0 |
| SRR28725322 | 67402 | 3603 | 37543 | 323 | 22853 | 29 | 59 | 69 | 123 | 156 | 22511 | 0 | 0 |
| SRR28725323 | 82689 | 4692 | 49913 | 404 | 29543 | 29 | 58 | 105 | 196 | 147 | 29047 | 0 | 0 |
| SRR28725324 | 61422 | 3438 | 34790 | 293 | 21231 | 27 | 54 | 55 | 96 | 39 | 20950 | 0 | 0 |

## 5. Классификация репрезентативных последовательностей

