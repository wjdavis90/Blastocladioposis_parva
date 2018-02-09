# Scripts used in analysis of Ion Torrent Data

The following Unix, perl, and macqiime scripts were used to analyze the Ion Torrent data. The sequences were generated on the same chip as Davis et al. 2018 (https://github.com/wjdavis90/-Inventory-of-chytrid-diversity-in-two-temporary-forest-ponds-using-a-multiphasic-approach); thus, there is overlap in the scripts and data. 

## Quality control

`$split_libraries.py -m mapping.txt -f seqs.fasta -q quality.qual -r -l 200 -L350 -M 1 -w 50 -g -b variable_length -o split_libraries -a 0 -H 6`

### Chimeras were detected using Usearch61. 

`$ ~/usearch6.1.544_i86osx32  ~/ITXseqs.fasta -uchime_ref -db ~/fungiLSU_train_041714.fasta  -strand plus -uchimeout results_ushime150721.txt -chimeras chimeras.fasta -nonchimeras nonchimeras.fasta`

### Chimeras were removed using macqiime.

`$ filter_fasta.py -f ~/ITXseqs.fasta -a ~/chimeras.fasta -n -o chimera_filtered_ITXseq.fasta`

## OTUs analysis

`$ pick_otus.py -i ~/chimera_filtered_ITXseq.fasta -m uclust -s 0.95 -o picked_otus_95`

Made a representative set of sequences.

`$pick_rep_set.py -i ~/chimera_filtered_ITXseq_otus.txt -m most_abundant -o rep_set.fasta`

## OTU tables were generated and filtered in macqiime.

`$ make_otu_table.py -i ~/chimera_filtered_ITXseq_otus.txt -t ~/rep_set_tax_assignments.txt -o all_otus_tax.biom`

The "all_otus_tax.biom" file contains data for both Davis et al. 2018 and *Blastocladiopsis parva*. I separated the relevant sample from the rest for downstream analyses.

`$ filter_samples_from_otu_table.py -i all_otus_tax.biom -m Mapping_ID_wm3b.txt -s 'Treatment:Bparva' -o ITX26_otu_table2.biom`

Then OTUs with less than 1 occurance were removed.

`$ filter_otus_from_otu_table.py -i ITX26_otu_table2.biom -n 1 -o ITX26_filtered.biom`

## Identifying putative *Blastocladiopsis parva* OTUS

The OTU table was converted to a list of OTUS.

`$ biom convert -b -i ITX26_filtered.biom -o ITX_otus3.txt`

NOTE: This use of `biom convert` is decapricated. It is recommended to use `biom convert --to-tsv -i -o` from henceforth. (http://biom-format.org/documentation/biom_conversion.html)

This list is converted into a space-delimited list.

`$ cat ITX26_otus3.txt | cut -f1 | tr "\n" " " > otus_to_perl.txt`

NOTE: Dependning on your environment, `sed 's/\n/ /g'` might be a better option.

`perl -ne 'if(/^>(\S+)/){$c=grep{/^$1$/}qw(denovo124 denovo126 denovo144 denovo219 denovo220 denovo232 denovo236 denovo310 denovo341 denovo396 denovo462 denovo519 denovo567 denovo568 denovo603 denovo737 denovo761 denovo830 denovo948 denovo952 denovo976 denovo1063 denovo1160 denovo1239 denovo1264 denovo1284 denovo1395 denovo1415 denovo1419 denovo1487 denovo1552 denovo1615 denovo1621 denovo1870 denovo1995 denovo2017 denovo2114 denovo2155 denovo2161 denovo2201 denovo2240 denovo2252 denovo2298 denovo2331 denovo2367 denovo2409 denovo2418 denovo2484 denovo2506 denovo2523 denovo2541 denovo2565 denovo2567 denovo2607 denovo2712 denovo2766 denovo2795 denovo2842 denovo2863 denovo2887 denovo2945 denovo2971 denovo3024 denovo3032 denovo3063 denovo3164 denovo3340 denovo3445 denovo3486 denovo3545 denovo3716 denovo3896 denovo3929 denovo3957 denovo4036 denovo4203 denovo4255 denovo4410 > )}print if $c' rep_set.fasta > ITX26.fasta`

This was run through the Ribosomal Database Project’s naïve-Bayesian classifier 2.7 which was retrained using a version of the RDP-NBC 28S fungal database No.11 that was hand curated to update the taxonomy of the chytrid sequences and to add additional chytrid sequences deposited in GenBank from taxonomic revisions and species descriptions. 

`java -Xmx1g -jar /rdp_classifier_2.7/dist/classifier.jar classify -t mytrain/rRNAClassifier.properties -o Classify_ITX26_OTUS.txt ITX26.fasta`

The resulting classification file was viewed in Excel. OTUS classified as Blastocladiomycota with >60% were selected and copied from the rep set.
