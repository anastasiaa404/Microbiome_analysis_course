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
```
input_reads=/home/anastasia/microbiome/qiime_test
manifest=/home/anastasia/microbiome/qiime_test/manifest.tmp

output_qza=/home/anastasia/microbiome/qiime_formatted/seq.qza
classifier=/home/anastasia/microbiome/silva138_AB_V4_classifier.qza
```

- Через nano исправить пути в файле manifest.tmp:
```
sample-id       absolute-filepath
SRR21066812     /home/anastasia/microbiome/qiime_test/SRR21066812.fastq
SRR21066823     /home/anastasia/microbiome/qiime_test/SRR21066823.fastq
SRR21066834     /home/anastasia/microbiome/qiime_test/SRR21066834.fastq
SRR21066845     /home/anastasia/microbiome/qiime_test/SRR21066845.fastq
SRR21066856     /home/anastasia/microbiome/qiime_test/SRR21066856.fastq
```
## 3. Создание seq.qza
```
qiime tools import --type 'SampleData[SequencesWithQuality]' --input-path $manifest --output-path $output_qza --input-format SingleEndFastqManifestPhred33V2
```

## 4. Обрезка и кластеризация с помощью DADA2 или DEBLUR