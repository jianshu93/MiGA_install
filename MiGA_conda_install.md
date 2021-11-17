### Install MiGA command line version for local computers/servers (for linux and MacOS)
1. Install miniconda3 and create miga conda environment.
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod +x ./Miniconda3-latest-Linux-x86_64.sh
bash ./Miniconda3-latest-Linux-x86_64.sh -b
### restart your terminal
conda create -y -n miga python=3.7
conda activate miga
### add hook to miga setup file
echo 'eval "$(conda shell.bash hook)" && conda activate miga' > ~/.miga_modules
```

2. Install R and ruby for the miga environment, ruby must be version 2.7
```
conda install -c conda-forge r-essentials=4.1
conda install -c conda-forge ruby=2.7
conda install sqlite
. ~/.miga_modules && ruby --version && R --version
```

3. Install depencencies in conda step by step
```
conda install -c conda-forge -c bioconda falco seqtk fastp hmmer barrnap
conda install -c conda-forge -c bioconda mcl bedtools blast prodigal diamond
conda install -c bioconda krona=2.8
### install newest version of miga via gem. gem should be installed automatically after installing ruby
gem install miga-base
```

4. Install dependencies that are not maintained in conda and cause conflicts
```
### faQCs, go to the github yourself and then compile it. Add binary to conda bin path then (which python will tell you the path)

git clone https://github.com/LANL-Bioinformatics/FaQCs.git
cd FaQCs
cp ./bin/Linux_x86_64_static/FaQCs ~/miniconda3/envs/miga/bin/


## fastani
wget https://github.com/ParBLiSS/FastANI/releases/download/v1.33/fastANI-Linux64-v1.33.zip
unzip fastANI-Linux64-v1.33.zip
cp ./fastANI ~/miniconda3/envs/miga/bin/

### idba_ud

### create a new Conda environment and install idba, then copy the idba_ud path to miga env path

conda create -y -n idba python=3.7
conda activate idba
conda install -c bioconda idba
which idba_ud

## put the which idba_ud path to miga env bin
cp ~/miniconda3/envs/idba/bin/idba_ud ~/miniconda3/envs/miga/bin/
conda activate miga

```


5. Install MyTaxa

```
git clone https://github.com/luo-chengwei/MyTaxa.git
cd MyTaxa
make

## on MacOS /usr/bin/python can be used
python2 ./utils/download_db.py
tar xzvf db.latest.tar.gz
rm db.latest.tar.gz
echo 'export PATH="'$PWD':$PATH"' >> ~/.miga_modules
```

6. Initiate MiGA and Download database
```
## will download all database by default but on PACE will fail downloading the AllGenomes.faa.dmnd because of home directory space limit. See Step 7 for how to fix
miga init --auto
```

7. Add database to path

```
cd ~/.miga_db
echo 'export PATH="'$PWD':$PATH"' >> ~/.miga_modules

cd MyTaxa

#### in PACE, AllGenomes.faa.dmnd in ~/.miga_db directory is symlinked to ~/shared-p/apps/MyTaxa/AllGenomes.faa.dmnd, you do not need this step for linux/macOS. For other linux server with limited home directory disk space, you can run miga init --auto locally to have the file AllGenomes.faa.dmnd

ln -s ~/shared-p/apps/MyTaxa/AllGenomes.faa.dmnd ~/.miga_db/AllGenomes.faa.dmnd



### link to MyTaxa folder
ln -s ~/.miga_db/AllGenomes.faa.dmnd ./AllGenomes.faa.dmnd
. ~/.miga_modules
## check again for database, ruby must be version 2.7, reinstall it via conda-forge if the version is not correct: conda install -c conda-forge ruby=2.7
miga init --auto
```

8. Testing using know genome in torque/slurm based supercomputer in interactive environment.
```
### on torque based job manage system

qsub -I -l nodes=1:ppn=24 -l walltime=10:00:00 -l mem=60gb -q inferno -A GT-ktk3-CODA20 -N interactive

### on Slurm based job manage system
srun --partition=ieg_128g,ieg_lm,ieg_64g --nodes=1 --ntasks-per-node=1 --cpus-per-task=24 --time=12:00:00 --mem=60G --pty bash -i


### test using 2 example fasta files, can be any genome downloaded or binned from metagenomes, if you see pdf files in try_out/mytaxa_scan then it looks good, it takes 7 minutes for 2 genomes, 12 threads for each genome
conda activate miga
time miga quality_wf --mytaxa-scan -c -t 12 -j 2 -o try_out ./*.fasta
```

