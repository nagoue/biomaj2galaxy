# BioMAJ2Galaxy

[![Build](https://travis-ci.org/genouest/biomaj2galaxy.svg?branch=master)](https://travis-ci.org/genouest/biomaj2galaxy)

## About

With this python module you can perform the following actions on a Galaxy server:

* Add new items to data libraries
* Remove items from data libraries
* Add new items to tool data tables using data managers
* Remove items from tool data tables

These scripts are primarily designed to be used as BioMAJ (http://biomaj.genouest.org) post processes,
but they can probably used directly from the command line if you need to.

This work has been published in GigaScience:

[BioMAJ2Galaxy: automatic update of reference data in Galaxy using BioMAJ.
Bretaudeau A, Monjeaud C, Le Bras Y, Legeai F, Collin O.
Gigascience. 2015 May 9;4:22. doi: 10.1186/s13742-015-0063-8. eCollection 2015.](https://dx.doi.org/10.1186%2Fs13742-015-0063-8)

## Installation

In most cases, you can install it easily with:

```bash
$ pip install biomaj2galaxy

# On first use you'll need to create a config file to connect to the Galaxy server, just run:
$ biomaj2galaxy init
Welcome to BioMAJ2Galaxy
url: http://localhost/
apikey: your-api-key
```

You need an API key to access your galaxy server. You need to use one from an admin account.

`allow_library_path_paste` should be set in `config/galaxy.yml` (or `config/galaxy.ini` for older versions)

Finally, if you want to add or remove items from tool data tables, you will need to install two data managers from the ToolShed:

 - `data_manager_manual` by the user `iuc`
 - `data_manager_fetch_genome_dbkeys_all_fasta` by the user `devteam`

## Usage

To see how to use each script (not necessarily from BioMAJ), just launch it with --help option.

Here is an example post-process and remove process configuration for BioMAJ, using data manager:

```
B1.db.post.process=GALAXY
GALAXY=galaxy_dm

galaxy_dm.name=galaxy_dm
galaxy_dm.desc=Add files to Galaxy tool data tables
galaxy_dm.type=galaxy
galaxy_dm.exe=add_galaxy_data_manager.py
galaxy_dm.args=-d "${localrelease}" -n "Homo sapiens (${remoterelease})" -g fasta/all.fa --bowtie2 bowtie/all --blastn blast/Homo_sapiens-ncbi_testing

db.remove.process=RM_GALAXY
RM_GALAXY=rm_galaxy_dm

rm_galaxy_dm.name=rm_galaxy_dm
rm_galaxy_dm.desc=Remove from Galaxy tool data tables
rm_galaxy_dm.type=galaxy
rm_galaxy_dm.exe=remove_galaxy_data_manager.py
rm_galaxy_dm.args="${db.name}-${removedrelease}" blastdb bowtie2_indexes
```

And the same using data libraries:

```
B2.db.post.process=GALAXY
GALAXY=galaxy_dl

galaxy_dl.name=galaxy_dl
galaxy_dl.desc=Add files to Galaxy data libraries
galaxy_dl.type=galaxy
galaxy_dl.exe=add_galaxy_library.py
galaxy_dl.args=-l "Homo sapiens genome (${remoterelease})" --lib-desc "Genome of Homo sapiens (version ${remoterelease}) downloaded from NCBI" -f "${localrelease}" --replace fasta/all.fa

db.remove.process=RM_GALAXY
RM_GALAXY=rm_galaxy_dl

rm_galaxy_dl.name=rm_galaxy_dl
rm_galaxy_dl.desc=Remove from Galaxy data libraries
rm_galaxy_dl.type=galaxy
rm_galaxy_dl.exe=remove_galaxy_library.py
rm_galaxy_dl.args=-l "Homo sapiens genome (${remote.release})" -f "${db.name}-${removedrelease}"
```

By default, relative file paths will be interpreted as relative to `${data.dir}/${dir.version}/${localrelease}` if these envionment variables are set. This can be disabled by using the --no-biomaj-env option.
