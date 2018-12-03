# Docker files for REPET package

The REPET package is a software suite dedicated to detect, classify (TEdenovo pipeline) and annotate (TEannot pipeline) repeats, specifically designed for transposable elements (TEs) in genomic sequences. Website: https://urgi.versailles.inra.fr/Tools/REPET

Since the package has many programs and needs a complicated installation to run,
here we provide the files and images to quickly deploy a container.

## Preparation

There are 3 important directories to choose:
* dockerdirectory: where you download this repo
* workdirectory: where the data is installed and processed
* mysqldir: where the persistent database from the pipeline is stored

Check that mysqldir must be outside workdirectory.

Make sure *dockerdirectory* has a complete permission schema (should be like this
most times) and it is not for example a mounted NTFS partition, for example, files
my.cnf and mysqld.cnf should look like this

```console
  -rw-r--r--  1 user         49 dic  1 17:43 my.cnf
  -rw-r--r--  1 user        486 dic  1 17:41 mysqld.cnf
```
with *-rw-r--r--* and **NOT -rwxrwxrwx**

```console
-rwxrwxrwx 1 root root  49 dic  2 21:29 my.cnf
-rwxrwxrwx 1 root root 486 dic  2 21:29 mysqld.cnf
```

Check that *workdirectory*  has at least 3GB plus your genome/s file/s sizes.

### Download Databases in {workdirectory}
```console
wget https://urgi.versailles.inra.fr/download/repet/ProfilesBankForREPET_Pfam27.0_GypsyDB.hmm
```

Downloads with registration:
* https://www.girinst.org/downloads/software/censor/ -> ver 4.2.29
* https://www.girinst.org/server/RepBase/index.php -> RepBase20.05_REPET.embl.tar.gz
* https://www.girinst.org/server/RepBase/index.php -> RepBaseRepeatMaskerEdition-20181026.tar.gz

The {workdirectory} should look like this:
```console
total 1343444

-rwxrwxr-x 1 user user   24266522 dic  1 11:30 censor-4.2.29.tar.gz
-rwxrwxr-x 1 user user 1250399747 dic  1 11:30 ProfilesBankForREPET_Pfam27.0_GypsyDB.hmm
-rwxrwxr-x 1 user user   44925658 dic  1 11:30 RepBase20.05_REPET.embl.tar.gz
-rwxrwxr-x 1 user user   56076961 dic  1 11:30 RepBaseRepeatMaskerEdition-20181026.tar.gz
```
### Configure processors and slots in sge.main

```console
# Example sge.main
qname                 main
[...]
processors            4
[...]
slots                 4
[...]
```

### Configure container directories (host machine)

In {dockerdirectory}/docker-compose.yml replace workdirectory
and mysqldir with absolute paths

```console
docker-compose.yml
  repet:
    volumes:
    - {workdirectory}:/out
  mysql:
    volumes:
      - {mysqldir}:/var/lib/mysql
```
```console
cp {mycontigs.fasta} {workdirectory}/proj.fa
```

### Start the services (host machine)
```console
docker-compose up --force-recreate
```

### Open a bash terminal to work
```console
docker-compose exec  -u sgeuser repet bash
#sgeuser@{someid}:/out
```
From now on, all the work is executed in this console: **sgeuser@{someid}:/out**

### Apply jobs and processors configuration
This has to be done **every** time container you run the previous step
```console
sudo -E /opt/sge/bin/lx-amd64/qconf -dq main
sudo -E /opt/sge/bin/lx-amd64/qconf -Aq /root/sge_queue.conf
```

### Config files and permissions
```console
sudo chown -R sgeuser:sgeuser /out
cp /docker_cfg/*.cfg /out
```

### Unzip downloaded files
```console
tar xfv RepBase20.05_REPET.embl.tar.gz
mv RepBase20.05_REPET.embl/* ./
rmdir RepBase20.05_REPET.embl
tar xfv RepBaseRepeatMaskerEdition-20181026.tar.gz
```

### Configure RepeatMasker
```console
cd /out
sudo cp -r /programs/RepeatMasker ./
sudo mv Libraries/* /out/RepeatMasker/Libraries
rmdir Libraries
sudo chown -R sgeuser:sgeuser /out/RepeatMasker

cd /out/RepeatMasker
perl configure
# ENTER
# **PERL PROGRAM** -> env
# Enter path [ xx ]: env
# **REPEATMASKER INSTALLATION DIRECTORY**
# Enter path [ xx ]: /out/RepeatMasker
# **TRF PROGRAM**
# Enter path [ xx ]: /usr/local/bin/trf
# Add a Search Engine:
# Enter Selection: 2
# **RMBlast (rmblastn) INSTALLATION PATH**
# Enter path [  ]: /usr/bin
# search engine for Repeatmasker? (Y/N)  [ Y ]: Y
# Enter Selection: 5
```

### Install CENSOR

```console
cd /out
tar xfv censor-4.2.29.tar.gz
cd censor-4.2.29
/configure --prefix=/out/censor_bin
make
make install
```

### Cleanup
```console
cd /out
rm RepBaseRepeatMaskerEdition-20181026.tar.gz RepBase20.05_REPET.embl.tar.gz censor-4.2.29.tar.gz censor-4.2.29
```

## Run the pipeline

## DeNovo

```console
cd /out

nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 1 | tee s1.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 2 -s Blaster | tee dn_s2.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 2 --struct | tee dn_s2s.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 3 -s Blaster -c Grouper | tee dn_s3g.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 3 -s Blaster -c Recon | tee dn_s3r.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 3 -s Blaster -c Piler | tee dn_s3p.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 3 --struct | tee dn_s3s.log

nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 4 -s Blaster -c Grouper -m Map | tee dn_s4g.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 4 -s Blaster -c Recon -m Map | tee dn_s4r.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 4 -s Blaster -c Piler -m Map | tee dn_s4p.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 4 --struct -m Map  | tee s4s.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 5 -s Blaster -c GrpRecPil -m Map --struct  | tee dn_s5.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 6 -s Blaster -c GrpRecPil -m Map --struct  | tee dn_s6g.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 7 -s Blaster -c GrpRecPil -m Map --struct | tee dn_s7g.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 8 -s Blaster -c GrpRecPil -m Map -f Blastclust --struct  | tee dn_s8b.log
nohup TEdenovo.py -P proj -C TEdenovo.cfg -S 8 -s Blaster -c GrpRecPil -m Map -f MCL --struct | tee dn_s8m.log

##ln proj_Blaster_GrpRecPil_Struct_Map_TEclassif_Filtered/proj_denovoLibTEs_filtered.fa proj1237_refTEs.fa
ln proj_Blaster_GrpRecPil_Struct_Map_TEclassif_Filtered_MCL/proj_denovoLibTEs_filtered_MCL.fa proj1237_refTEs.fa
```
### DeNovo post processing
```console
CreateGFF3sForClassifFeatures.py -C CreateGFF3sForClassifFeatures.cfg -f proj1237_refTEs.fa -v 3

cd Visualization_Files/;
mkdir gff_reversed
cut -f1,2 ../proj_Blaster_GrpRecPil_Struct_Map_TEclassif_Filtered/classifFileFromList.classif > proj_denovoLibTEs_filtered.len
for file in `ls *.gff3`;
do
grep -P "^#" $file > gff_reversed/$file;
while read TE len;
do gawk -F"\t" '{if($1 ~ /_reversed/ && $1 ~ /'$TE'/){rstart='$len'-$5+1;rend='$len'-$4+1; if($7 ~ /+/){rstr="-"}; if($7 ~ /-/){rstr="+"};OFS="\t";print $1,$2,$3,rstart,rend,$6,rstr,$8,$9}else{if($1 ~ /'$TE'/){print $0}}}' $file;
done < proj_denovoLibTEs_filtered.len >> gff_reversed/$file;
done
cd /out
```

### Annotation 1st round
```console

ln proj.fa proj1237.fa
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 1 | tee ta1_s1.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 2 -a BLR | tee ta1_s2b.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 2 -a RM | tee ta1_s2r.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 2 -a CEN | tee ta1_s2c.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 2 -a BLR -r | tee ta1_s2br.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 2 -a RM -r | tee ta1_s2rr.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 2 -a CEN -r | tee ta1_s2cr.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 3 -c BLR+RM+CEN | tee ta1_s3.log
nohup TEannot.py -P proj1237 -C TEannot1237.cfg -S 7 | tee ta1_s7.log

PostAnalyzeTELib.py -a 3 -p proj1237_chr_allTEs_nr_join_path -s proj1237_refTEs_seq -g {genomesize}
GetSpecificTELibAccordingToAnnotation.py -i proj1237_chr_allTEs_nr_join_path.annotStatsPerTE.tab -t proj1237_refTEs_seq
ln proj1237_chr_allTEs_nr_join_path.annotStatsPerTE_FullLengthFrag.fa proj_refTEs.fa
```

### Annotation 2nd rount
```console
nohup TEannot.py -P proj -C TEannot.cfg -S 1 | tee ta2_s1.log
nohup TEannot.py -P proj -C TEannot.cfg -S 2 -a BLR | tee ta2_s2b.log
nohup TEannot.py -P proj -C TEannot.cfg -S 2 -a RM | tee ta2_s2r.log
nohup TEannot.py -P proj -C TEannot.cfg -S 2 -a CEN | tee ta2_s2c.log
nohup TEannot.py -P proj -C TEannot.cfg -S 2 -a BLR -r | tee ta2_s2br.log
nohup TEannot.py -P proj -C TEannot.cfg -S 2 -a RM -r | tee ta2_s2rr.log
nohup TEannot.py -P proj -C TEannot.cfg -S 2 -a CEN -r | tee ta2_s2rc.log
nohup TEannot.py -P proj -C TEannot.cfg -S 3 -c BLR+RM+CEN | tee ta2_s3.log
nohup TEannot.py -P proj -C TEannot.cfg -S 4 -s TRF | tee ta2_s4t.log
nohup TEannot.py -P proj -C TEannot.cfg -S 4 -s Mreps | tee ta2_s4m.log
nohup TEannot.py -P proj -C TEannot.cfg -S 4 -s RMSSR | tee ta2_s4r.log
nohup TEannot.py -P proj -C TEannot.cfg -S 5 | tee ta2_s5.log
nohup TEannot.py -P proj -C TEannot.cfg -S 6 -b tblastx | tee ta2_s6t.log
nohup TEannot.py -P proj -C TEannot.cfg -S 6 -b blastx | tee ta2_s6b.log
nohup TEannot.py -P proj -C TEannot.cfg -S 7 | tee ta2_s7.log


TEannot.py -P proj -C TEannotplot.cfg -S 8 -o GFF3
cat proj_GFF3chr/*.gff3 |grep -v "##" > proj_refTEs.gff
PostAnalyzeTELib.py -a 3 -g {genomesize} -p proj_chr_allTEs_nr_noSSR_join_path -s proj_refTEs_seq -v 2 #1303001
mkdir plotCoverage
python /repet/SMART/Java/Python/plotCoverage.py -i proj_refTEs.gff -f gff3 -q proj_refTEs.fa --merge  -l grey -o plotCoverage/proj
TEannot.py -P proj -C TEannot.cfg -S 8 -o GFF3

exit
```

### Change ownership of the files (host machine)
```console
sudo chown -R $USER:$USER {workdirectory}
```


### Waiting processes
Some jobs never start if in the table jobs exists an entry in 'waiting' status.
```console
docker-compose exec mysql bash  # Host
mysql                           # Container
use repet                       # Container Mysql
DELETE FROM jobs;               # Container Mysql
```

## References
* https://urgi.versailles.inra.fr/Tools/REPET/INSTALL
* https://biosphere.france-bioinformatique.fr/wikia2/index.php/REPET_practical_course_urgi
* https://urgi.versailles.inra.fr/Tools/REPET/TEdenovo-tuto
* https://urgi.versailles.inra.fr/Tools/REPET/TEannot-tuto
