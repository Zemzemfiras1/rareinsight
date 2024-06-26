# RareDisease Reference files

# Get GitHub repository (Thank you Marius Bjørnstad for this resource!)
https://github.com/fa2k/raredisease-configs/archive/refs/heads/main.zip
unzip master.zip

# igenomes GRCh38
https://s3.amazonaws.com/igenomes.illumina.com/Homo_sapiens/NCBI/GRCh38/Homo_sapiens_NCBI_GRCh38.tar.gz

tar xf Homo_sapiens_NCBI_GRCh38.tar.gz

# ExpansionHunter / Stranger catalogue file
https://raw.githubusercontent.com/Clinical-Genomics/stranger/master/stranger/resources/variant_catalog_grch38.json

# VEP cache (v107)
https://ftp.ensembl.org/pub/release-107/variation/indexed_vep_cache/homo_sapiens_merged_vep_107_GRCh38.tar.gz

tar xf homo_sapiens_merged_vep_107_GRCh38.tar.gz
change name to vep_cache

# VEP plugins (v107)

At least the plugin pLI is required in order to run the SNV annotation (ADD_MOST_SEVERE_PLI). Other plugins are definitely needed to run the ranking steps in GENMOD processes. Plugins are installed into the vep_cache dir using the following script. Internet access and singularity is required.

bash install-vep-plugins.sh # Change script to correct file paths

(https://raw.githubusercontent.com/Ensembl/VEP_plugins/release/107/pLI_values.txt, https://raw.githubusercontent.com/Ensembl/VEP_plugins/release/107/LoFtool_scores.txt, http://hollywood.mit.edu/burgelab/maxent/download/fordownload.tar.gz)

Place these in vep_cache dir. The vep config ext.args is defined in the workflow repo <raredisease>/conf/modules/annotate_snvs.config. The override in medGenConfigs/process-overrides.conf sets the following paths.

spliceai_scores.raw.indel.hg38.vcf.gz
spliceai_scores.raw.indel.hg38.vcf.gz.tbi
spliceai_scores.raw.snv.hg38.vcf.gz
spliceai_scores.raw.snv.hg38.vcf.gz.tbi

# Intervals files
Create bed files and interval_lists from genome dict file using the following script:

bash create-bed-and-intervals.sh


# genmod - rank_model_snv.ini / rank_model_sv.ini
Initially copied from the test datasets and Clinical Genomics's configs (https://github.com/Clinical-Genomics/reference-files/tree/master/rare-disease/rank_model).

https://github.com/Clinical-Genomics/reference-files/blob/master/rare-disease/rank_model/rank_model_-v1.36-.ini

https://github.com/Clinical-Genomics/reference-files/blob/master/rare-disease/rank_model/svrank_model_-v1.11-.ini

# Clinvar
https://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh38/clinvar.vcf.gz

https://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh38/clinvar.vcf.gz.tbi

The ClinVar files have annotations in the latin1 charset. There are characters outside the generally compatible ASCII subset, like é, which upsets the python scripts used in the pipeline. The script convert-clinvar-charset.sh converts the vcf file to UTF-8. The converted file is used in the pipeline.

# CADD

CADD doesn't like chr? chromosome names. I've fixed it in a hacky way; needs to be done properly but I don't have much time now Replace the entire cadd process file:

cp -b scripts/cadd-process-fix-main.nf ~/nf-core-raredisease_1.1.1/1_1_1/modules/nf-core/cadd/main.nf

Furthermore the pipeline seems to multiply the number of TABIX_VEP jobs to the split number squared. I don't know if it's a bug or I'm doing something wrong. The subworkflow was modified to use only CADD outputs into VEP:

cp -b scripts/doctored-annotate_snvs.nf ~/nf-core-raredisease_1.1.1/1_1_1/subworkflows/local/annotate_snvs.nf

# Indels
https://krishna.gs.washington.edu/download/CADD/v1.6/GRCh38/annotationsGRCh38_v1.6.tar.gz (205GB)

tar xf annotationsGRCh38_v1.6.tar.gz

# SNVs
https://krishna.gs.washington.edu/download/CADD/v1.6/GRCh38/whole_genome_SNVs.tsv.gz (81GB)

https://krishna.gs.washington.edu/download/CADD/v1.6/GRCh38/whole_genome_SNVs.tsv.gz.tbi

# Create new index for SNVs
There have been some issues with the default tbi index. We create a csi index manually using > tabix- make-cadd-index.sh

# vcfanno
vcfanno_resources needs absolute paths - edit file to have correct path

# svdb
edit ssvdb_querydb_files.csv from test data to have correct path

# gnomAD
Gnomad mitochondrial data file is tiny and doesn't need to be reformatted. It is used by vcfanno. Downloaded from:
https://storage.googleapis.com/gcp-public-data--gnomad/release/3.1/vcf/genomes/gnomad.genomes.v3.1.sites.chrM.vcf.bgz
and
https://storage.googleapis.com/gcp-public-data--gnomad/release/3.1/vcf/genomes/gnomad.genomes.v3.1.sites.chrM.vcf.bgz.tbi

https://gnomad.broadinstitute.org/downloads#v3

The script make-gnomad-af-file.sh creates a combined VCF file
  
# gnomAD tab file
The gnomad_af argument expects a tab.gz file, not a VCF. Run:
bash convert-gnomad-to-tsv.sh

You also need an index of the tab file:

```
tabix -p vcf gnomad.genomes.v3.1.2.sites.af.tab.gz
```
This file can then be specified as --gnomad_af_idx in the yml params file.

# gnomAD SV
https://ftp.ncbi.nlm.nih.gov/pub/dbVar/data/Homo_sapiens/by_study/vcf/nstd166.GRCh38.variant_call.vcf.gz

ttps://ftp.ncbi.nlm.nih.gov/pub/dbVar/data/Homo_sapiens/by_study/vcf/nstd166.GRCh38.variant_call.vcf.gz.tbi

