# Scripts used in analysis of Ion Torrent Data
The following Unix, perl, and macqiime scripts were used to analyze the Ion Torrent data.
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
### Taxonomy was assigned with the RPD classifier.
`$java -Xmx1g -jar ~/rdp_classifier_2.7/dist/classifier.jar ~/mytrained/rRNAClassifier.properties -o classify.txt ~/rep_set.fasta'''

The output file rep_set_tax_assignments.txt generated with assign_taxonomy.py was edited to reflect the RDP classification.
## OTU tables were generated and filtered in macqiime.
`$ make_otu_table.py -i ~/chimera_filtered_ITXseq_otus.txt -t ~/rep_set_tax_assignments.txt       -o all_otus_tax.biom`
### Potential contamination was removed using the positive control sample.
First, an OTU table with only the OTUs appearing in the positive control was created.
`$filter_samples_from_otu_table.py -i ~/all_otus_tax.biom -m mapping.txt -s ‘Treatment:Control’ -o otu_table_control_only.biom`

The control OTU table was filtered to remove an OTU with an abundance <1.
`$ filter_otus_from_otu_table.py -i otu_table_control_only.biom -n 1 -o filtered_otu_table_control.biom`

The control OTU table was converted to a list of OTUs.
`$ biom convert -b -i filtered_otu_table_control.biom -o otus_to_remove.txt`

The list was then used to remove potential contaminants from the full OTU set.
`$ filter_otus_from_otu_table.py -i ~/ all_otus_tax.biom -e otus_to_remove.txt -o filtered_all_otus.biom`

The OTU was reduced to just the Blastocladiomycota sequences.
`$ filter_taxa_from_otu_table.py -i ~/filtered_otus_oct_dec.biom -p D_3__Blastocladiomycota -o filtered_blasto_otus_oct_dec.biom`

## Converted OTU table into fasta files.
`$biom convert -b -i ~/ filtered_blasto_otus_oct_dec.biom -o put_Bparva_otus.fasta`
