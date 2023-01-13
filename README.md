# SARS-CoV-2 build definitions for Africa CDC

Build configuration for Africa CDC SARS-CoV-2 builds hosted at https://nextstrain.org/groups/africa-cdc.

## Instructions for running Africa CDC builds with GISAID data

### Setup Repositories

```
git clone https://github.com/nextstrain/ncov-africa-cdc.git
git clone https://github.com/nextstrain/ncov.git
cd ncov
cp -r ../ncov-africa-cdc/africa_cdc_profile .
```

### Download data

Login to [GISAID (gisaid.org)](https://www.gisaid.org/) and select the “EpiCoV” link from the navigation.

Select the “Downloads” link from the EpiCoV navigation bar. Scroll to the “Genomic epidemiology” section and select the “nextregions” button. Select the “Africa” button. Save the file as `hcov_africa.tar.gz` in the `ncov/data/` workflow directory.

Click "Back" to return to the main "Download" dialog, find the “Download packages” section, and select the “FASTA” button. Save the full GISAID sequences as `data/sequences_fasta.tar.xz`.

Select the “metadata” button from that same “Download packages” section and download the corresponding file as `data/metadata_tsv.tar.xz`.

### Filter data

Navigate to the `ncov` workflow directory and enter a Nextstrain runtime shell there.

```
nextstrain shell .
```

Extract African metadata and sequences from full GISAID downloads.

```
# Get metadata for Africa directly from tarball.
tar xOf data/metadata_tsv.tar.xz metadata.tsv \
  | tsv-filter -H --str-in-fld Location:Africa \
  | xz -c -2 > data/metadata_africa.tsv.xz

# Get strain names for genomes.
# GISAID uses virus name, collection date, and submission date
# delimited by a pipe character.
xz -c -d data/metadata_africa.tsv.xz \
  | tsv-select -H -f 'Virus\ name','Collection\ date','Submission\ date' \
  | sed 1d \
  | awk -F "\t" '{ print $1"|"$2"|"$3 }' > data/strains_africa.txt

# Get genomes for strain names from tarball.
tar xOf data/sequences_fasta.tar.xz sequences.fasta \
  | seqkit grep --by-name -f data/strains_africa.txt \
  | xz -c -2 > data/sequences_africa.fasta.xz
```

### Run builds locally

Navigate to the `ncov` workflow directory; these instructions assume this is a sibling directory to this repository.
By default, the following command will run builds for all countries and regions.

```bash
nextstrain build \
  --cpus 4 \
  --memory 8Gib \
  . \
  --configfile ../ncov-africa-cdc/builds_africa.yaml
```

You can specify a comma-delimited subset of builds to run by build name.
The following example only runs builds for Ghana and Kenya.

```bash
nextstrain build \
  --cpus 4 \
  --memory 8Gib \
  . \
  --configfile ../ncov-africa-cdc/builds_africa.yaml \
  --config active_builds=ghana,kenya
```

### Run builds on AWS Batch

To run builds on AWS Batch, the build config and profiles directory must be in the `ncov` workflow directory.
Sync those files into `ncov`.

```bash
# Run from ncov/
rsync -arvz \
  ../ncov-africa-cdc/builds_africa.yaml \
  ../ncov-africa-cdc/africa_cdc_profile \
  .
```

Run all builds on AWS Batch.

```bash
nextstrain build \
  --aws-batch \
  --cpus 36 \
  --memory 70Gib \
  . \
  --configfile builds_africa.yaml
```

### Run builds on Terra

1. **Terra** is a platform for biomedical research. To learn more and register an account, please follow the [Getting Started Guide](https://terra.bio/resources/getting-started/). 

2. To run builds on Terra, you need to upload the build config and profiles directory. To do this, download this repository and zip the profile folder.

  ```
  git clone git clone https://github.com/nextstrain/ncov-africa-cdc.git
  cd ncov-africa-cdc
  zip -r africa-cdc-profile.zip africa-cdc-profile
  
  # Upload these files to Terra
  ls -l africa-cdc-profile.zip
  ls -l builds_africa.yaml
  ```

  Then, upload the files to Terra using the [Data Uploader](https://support.terra.bio/hc/en-us/articles/4419428208411-Data-Uploader).

3. Terra allows you to run parallel jobs from a **Table**. A template TSV file is provided in this repository which can be used to create a Terra table by following [these instructions](https://support.terra.bio/hc/en-us/articles/360059242671).

  ```
  # TSV for creating a table in Terra
  ls -ltr builds.tsv
  ```

4. [**Dockstore**](https://dockstore.org/about) is a platform for sharing reusable and scalable analytical tools and workflows. To import the `ncov` and `gisaid_ingest` workflows from Dockstore, select "Terra" under "Launch with":

  * [Dockstore: nextstrain/ncov/gisaid_ingest](https://dockstore.org/workflows/github.com/nextstrain/ncov/gisaid_ingest:master?tab=info)
  * [Dockstore: nextstrain/ncov/ncov](https://dockstore.org/workflows/github.com/nextstrain/ncov/ncov:master?tab=info)

5. To learn more about running any workflow in Terra, please visit [Terra's Getting started running workflows](https://support.terra.bio/hc/en-us/articles/360036379771-Get-started-running-workflows).


TBD.