#Data quality checking with FastQC

####Information in this tutorial is based on the FastQC manual which can be accessed [here](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/).

FastQC is a relatively quick and non labor-intesive way to check the quality of your NGS data.

Before starting, we need to make sure we have sequencing files that end in .fastq

FastQC is already loaded and active in our AMI, so all we have to do is use the 'fastqc' command:
```
fastqc C02_05102014_R1_D03_TTACTGTGCGAT.fastq
```
This will create two new files with the same name and the extensions `.fastqc.zip` and `fastqc.html`. As you may be able to guess, these are processed files in zip and html format.

Using scp, transfer the html file to your desktop. Now double-click on the file and it should open in your browser.
On the left-hand side of the screen, there will be a summary of the analyses with some combination of green checkmarks, yellow exclamation points, and red Xs, depending on whether or not the sequences pass the quality check for each module.


###1: Basic Statistics
![basic statistics](../img/basic_statistics.jpg)

Basic statistics displays a chart containing information about your file, including the name, how many reads were analyzed, and whether or not any of the reads were flagged for poor quality. In this case, we had 197235 sequences. None of them were flagged as poor quality. The average sequence length was 253 bases, with 55% GC content.

###2: Per base sequence quality
![per base sequence quality](../img/per_base_sequence_quality.jpg)

Per base sequence quality shows the quality of each sequence at every position from 1 to 253, in this case. It displays this information using box and whisker plots to give a sense of how much variation there was among the reads. Ideally we would want the entire plot to be contained within the green region; this would be considered very good quality. While having part of the plot in the orange or red regions is not preferable, sequences can still pass the quality check with lower quality, as in our example. When the lower quartile for any position is less than 10 or the median is less than 25, the module will give a warning. When the lower quartile for any position is less than 5 or the median is less than 20, the sequence will fail this quality check.

###3: Per tile sequence quality
![per tile sequence quality](../img/per_tile_sequence_quality.jpg)

Per tile sequence quality is a heatmap display of the flow cell quality by individual tiles. If the figure is a solid bright blue, the flow cell tile quality was consistently great! If there are patches of lighter blue or any other color, there was a problem associated with one of the tiles (such as a bubble or smudge) and this may correspond with a decrease in sequence quality in those regions. Above you can see light blue patches which indicate potential problems with the sequencing lane. However, in this case it is still good enough to pass the quality check; this would be more concerning if we had orange or red tiles.

###4: Per sequence quality scores
![per sequence quality scores](../img/per_sequence_quality_scores.jpg)

Per sequence quality scores represent the quality of each read. The plot is created using Phred scores, which are based on a logarithmic scale. A Phred score of 30 indicates an error rate of 1 base in 1000, or an accuracy of 99.9%, while a Phred score of 40 indicates an error rate of 1 base in 10,000, or an accuracy of 99.99%. Sequences will yield a warning for this module if there is an error rate of 0.2% or higher (Phred score below 27). Sequences will fail this quality check if they have an error rate of 1% or higher (Phred score below 20.)

###5: Per base sequence content
![per base sequence content](../img/per_base_sequence_content.jpg)

Per base sequence content shows, for each position of each sequence, the base composition as a percentage of As, Ts, Cs and Gs. In a typical genome we expect the percentages of each base to be roughly equal, so the lines should be parallel and within about 10% of each other. We all know that GC content varies between organisms though, so some variation is expected. This module will yield a warning if the base content varies more than 10% at any position, and a sample will fail if there is more than 20% variation at any position, as in the example above.

###6: Per sequence GC content
![per sequence GC content](../img/per_sequence_GC_content.jpg)

Per sequence GC content displays the GC content for all reads along with the "theoretical distribution" of GCs. The peak of the red line corresponds to the mean GC content for the sequences, while the peak of the blue line corresponds to the theoretical mean GC content. Your GC content should be normally distributed; shifts in the peak are to be expected since GC content varies between organisms, but anything other than a normal curve might be indicative of contamination. A warning is raised if sequences outside of the normal distribution comprise more than 15% of the total. A sample will fail if more than 20% of sequences are outside the normal distribution. Failures are usually due to contamination, frequently by adapter sequences.

###7: Per base N content
![per base N content](../img/per_base_N_content.jpg)

Per base N content shows any positions within the sequences which the which have not been called as A, T, C or G. Ideally the per base N content will be a completely flat line at 0% on the y-axis, indicating that all bases have been called. Samples receive a warning if the N content is 5% or greater, and will fail if N content is 20% or greater. Our example shows the ideal result for this module!

###8: Sequence length distribution
![sequence length distribution](../img/sequence_length_distribution.jpg)

This module simply shows the length of each sequence in the sample. Depending on the sequencing platform used, this will vary. For Illumina sequencing, each read should be the same size, with variation of one or two bases being acceptable. For other platforms, a relatively large amount of variation is normal. The module will show a warning if there is any variation in sequence length, which can be ignored if you know that this is normal for your data. A failure here means that at least one sequence had a length of 0. Our example passes this module as all of the sequences are 253 bp with no variation.

###9: Sequence duplication levels
![sequence duplication levels](../img/sequence_duplication_levels.jpg)

The sequence duplication levels plot shows the number of times a sequence is duplicated on the x-axis with the percent of sequences showing this duplication level on the y-axis. Normally a genome will have a sequence duplication level of 1 to 3 for the majority of sequences, with only a handful having a duplication level higher than this; the line should have an inverse log shape. A high duplication level for a large percentage of sequences is usually indicative of contamination. This module will issue a warning if more than 20% of the sequences are duplicated, and a failure if more than 50% of the sequences are duplicated. A warning or failure will occur if there PCR artifacts or enriched sequences; this is sometimes to be expected.

###10: Overrepresented sequences
![overrepresented sequences](../img/overrepresented_sequences.jpg)

If a certain sequence is calculated to represent more than 0.1% of the entire genome, it will be flagged as an overrepresented sequence and yield a warning for this module. The presence of sequences that represent more than 1% of the whole genome will result in a failure, as seen above.
A frequent source of "overrepresented sequences" is Illumina adapters, which is why it's a good idea to trim sequences before running FastQC. Another occasional source of overrepresented sequences is high copy-number plasmids, although they are frequently a result of contamination. FastQC checks for possible matches to overrepresented sequences, although this search frequently returns "no hit". However it is usually quite easy to identify the overrepresented sequences by doing a simple BLAST search.


###11: Adapter content
![adapter content](../img/adapter_content.jpg)
This module searches for specific adapter sequences. A sequence that makes up more than 5% of the total will cause a warning for this module, and a sequence that makes up more than 10% of the total will cause a failure. Our example shows no contamination with adapter sequences, which is ideal. If there were a significant number of adapter sequences present, we would want to use a trimming program to remove them before further analysis.



###12: Kmer content
![kmer content](../img/kmer_content.jpg)
![kmer content 2](../img/kmer_content_part_II.jpg)
In a completely random library, any kmers would be expected to be seen about equally in each position (from 1-253 in this case). Any kmers that are specifically enriched at a particular site are reported in this module. If a kmer is enriched at a specific site with a p-value of less than 0.01, a warning will be displayed. A failure for this module occurs if a kmer is enriched at a site with a p-value of less than 10^-5. It is relatively common to see enriched kmers near the beginning of a sequence, once again because of adapter sequences. The warning seen in the example is likely due to adapters based on the position of the kmers.


###Questions about FastQC can usually be answered with the [documentation](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/). Happy quality checking!