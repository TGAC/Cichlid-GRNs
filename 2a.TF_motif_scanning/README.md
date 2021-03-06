===============================================================================

The scripts described below pertain to the following publication:

Evolution of regulatory networks associated with traits under selection in cichlids 
Tarang Mehta; Christopher Koch; Will Nash; Sara Knaack; Padhmanand Sudhakar; Marton Olbei; Sarah Bastkowski; Luca Penso-Dolfin; Tamas Korcsmaros; Wilfried Haerty; Sushmita Roy; Federica Di Palma 
Genome Biology (2020)

Note: 
The scripts are in the process of being collated into a pipeline tool 
which will be published as a companion paper. 
As such, scripts lack comments within their body. Future versions will be fully
commented and released with full documentation

Author:
All scripts described below were written and tested by Dr. Will Nash
(ORCID: 0000-0002-6790-1167) during the preparation of material for this paper. 

License: 
This source code is freely available to all under the Creative Commons Attribution-ShareAlike license (CC BY-SA) and under the standard GPL 3.0 license from Github

The scripts below were written to run in the stated order, with each script
relying on the output of that prior to it. Where a and b versions exist, these are
motifs for a) human and b) mouse data and were handled separately
during the development of the approach. The scripts are identical, but deal with
species specific descriptors differently. Both versions are included here for
clarity, but scripts will be generalised in the publication version.

===============================================================================

Scripts:

- 1_GTRD_parse.py
- 2a_extrapSites_hum_v3.py
- 2b_extrapSites_mus_v3.py
- 3a_extrapSites_hum_v4.py
- 3b_extrapSites_mus_v4.py
- 4a_extrapSites_hum_v4ex.py
- 4b_extrapSites_mus_v4ex.py
- 5a_extrapSites_hum_afterMEME.py
- 5b_extrapSites_mus_afterMEME.py
- 6_TFBSextrapolation_v5.py
- 7_TFsortSplit5.py
- 8_run_mat_qual_nm.py
- 9a_matQualResultsParser1_hsap.py
- 9b_matQualResultsParser1_mus.py
- 10a_mat_qual_fimo_parse_hsap_wg.py
- 10b_mat_qual_fimo_parse_mmus_wg.py

===============================================================================

Descriptions:

1_GTRD_parse.py

    This script will extract the genomic sequences which fall under peaks
    in the GTRD databse. The output will be in tf:tg:tfbs_peak format tsv.

    The inputs are the GTRD database metaclusters that have been reduced using
    bedtools intersect to remove overlapping peaks and then bedtools closest to
    remove all non adjacent genes within a 10kb window of peaks. These two files
    respectively represent both forward and reverse strand. The model species
    genome in FASTA format is also input. A further input is the
    tfClass2ensembl.txt file from GTRD_17.04 (or more recent equivalent), this
    file recapitulates the link between the GTRD TF info and the ENSEMBL gene
    ids. This is essential for later parsing of orthology between model and
    non-model speices.

2a_extrapSites_hum_v3.py
2b_extrapSites_mus_v3.py

    This script substitutes binding sites extracted from the GTRD ChipSeq
    database with the core motif as described in the JASPAR2018 db, HOCOMOCO
    db, or Uniprobe topKmers list. JASPAR2018 is searched first, then HOCOMOCO,
    then the Uniprobe topKmers. For all searches, mismatches are allowed
    between db motif sequence and peak sequence, based on motif sequence kmer
    length (5mer: 2mm; 6-11mer: 3mm; 12-17mer: 4mm; 18+mer: 5mm).

    Input files are: parsed JASPAR hg38/mm10 input file (motifs that appear
    only in these species along with corresponding ENSEMBL ids, found using
    biodb2db server), parsed meta data describing the full JASPAR2018 database
    (downloaded from JASPAR2018 and parsed into single line format), the file
    path of a directory containing the '.sites' files downloaded from the
    JASPAR2018 file repository, the non-redundant TF:TG:TFBS file provided by
    1_GTRD_parse.py, the Uniprobe topKmers file, parsed from the meta data
    provided online, the HOCOMOCO data parsed into a non redundant single line
    format.

3a_extrapSites_hum_v4.py
3b_extrapSites_mus_v4.py

    This script uses the same sorting methods as the _v3 script, but adds motifs
    which are generated during the first pass analysis conducted by _v3 that are
    constructed for all 5 species. Such motifs were created when one single
    species had less than 3 TFBS sequences for a TF:TG pair.

    Inputs are the same as in extrapSites_v3, with species wide motifs recovered
    from the architecture created in _v3.

    This script will be redundant in the final tool.

4a_extrapSites_hum_v4ex.py
4b_extrapSites_mus_v4ex.py

    This script does the same as the _v4 but incorporates exceptions to the
    script above. These exceptions are TF binding sequences found using the
    species wide motifs generated above.

    Inputs are the same as in extrapSites_v4, with species wide motifs recovered
    from the architecture created in _v3.

    This script will be redundant in the final tool.

5a_extrapSites_hum_afterMEME.py
5b_extrapSites_mus_afterMEME.py

    This script follows the three preceding scripts (_v3, _v4, _v4ex), and given
    the final set of recovered motif sequences generated by these scripts,
    identifies TFs for which no TFBS core motif could be identified in any
    database, either at a species specific, or collated 5 species level. For
    these TFs, all peak sequences identified as associated with binding events of
    this TF are submitted to the MEME software for de-novo motif prediction.
    These de-novo predicted motifs are then used to scan whole genome promoters
    where motifs have not been discovered in any other part of the pipeline

    Inputs are the same as in extrapSites_v4, with species wide motifs recovered
    from the architecture created in _v3, _v4, and v4ex.

6_TFBSextrapolation_v5.py

    This script conducts the lift over of motifs from the curated versions
    generated by the 4 extrapSites scripts. The first process of the script is to
    create a  matrix of gene identifier which is based on strict orthology of
    both TF and TG between model and non-model species. This is specified by the
    orthology table input. When the full set of candidates is identified through
    this process, the promoter sequences input (from the non-model genome) for
    each TG is scanned for each of the input set of TFBS sequences for every TF
    associated with it. Each putative TFBS sequence is aligned against the
    promoter. To be added to the non-model species specific motif, it must match
    a sub-sequence of the promoter by a similarity threshold that is given as an
    input (Default setting used in this study is 70% - neutral divergence).

    Inputs are 1:1 Orthology file (; sep), Non model species promoter FASTA, TFBS
    set from model species (from extrapSites scripts), similarity threshold (0-1)

7_TFsortSplit5.py

    This script follows the TFBSextrapolation_v5.py script in the TFBS
    extrapolation from model to non-model genomes workflow. It will sort the
    output of TFBSextrapolation_v5.py by TF and then output a TF specific FASTA
    file with each TFBS motif sequence as an individual FASTA entry. The TF
    specific TFBS FASTA files are then be passed to the info-gibbs tool from the
    RSAT suite to be converted into matrices in the info-gibbs format (.ig). The
    convert-matrix tool from the RSAT suite is then used convert the matrix
    generated from info-gibbs format to transfac format (.transfac). Directories
    are created to store all of the files above.

    Inputs are: TFBSextrapolation_v5 output for species from specific model
    species (Hsap or Mmus in this case), orthology file (as used in
    TFBSextrapolation_v5.py), the input core TFBS motif sequences from the
    TFBSextrapolation_v5.py step (used to input a median motif length to the
    info-gibbs tool).

8_run_mat_qual_nm.py

    This script generates submission scripts to employ the matrix_quality.pl
    script from the RSAT suite to permute each position weight matrix (PWM)
    through a given number of random permutations (We used 1000) and to test the
    probability of finding a sequence matching this PWM in a given set of
    sequences (The non-model species promoters). The P-value generated can then
    be used in the promoter wide scanning conducted in the next script.

    Inputs are: directory of species level motifs in transfac format, non-model
    species promoter FASTA, number of matrix permutations, background file
    (generated by XXX), species identifier, directory of cichlid level motifs in
    transfac format.

9a_matQualResultsParser1_hsap.py
9b_matQualResultsParser1_mus.py

    This script parses the results from RSAT suite matrix_quality.pl script when
    run using the run_mat_qual_nm.py script above (with 1000 matrix permutations
    per motif) on given list of transcription factors. It will create a matrix of
    all P-values generated relative to each TF (at three levels: default, species
    specific, and all species wide) for use in FIMO scanning. This matrix is
    output, but also within the script, used to scan the promoters of each
    species using FIMO from the MEME suite.

    Inputs are: JASPAR File, Hsap JASPAR Endsembl, Mmus JASPAR Ensembl, Directory
    containing all of the TF specific RSAT results directories

    This script is heavily hard-coded to the Cichlid project and will be
    generalised to the format of model to non-model species extrapolation within
    the final tool. It also deploys the FIMO tool using a hard-coded PBS
    submission script which, again, will be removed in the final version.

10a_mat_qual_fimo_parse_hsap_wg.py
10b_mat_qual_fimo_parse_mmus_wg.py

    This script is a rapidly written parser which scans all the output of the
    genome wide FIMO scans of promoter sequences conducted by the
    matQualResultsParser1.py script. It simply concatenates the FIMO results of
    all TFs from a given motif source (JASPAR, Cichlid specific, Cichlid wide)
    into a single file per species. These files are stored in a new directory
    created by the script.

    Inputs is: File path of the output created by the matQualResultsParser1.py
    script.

    This script needs to be heavily rewritten to a readable format, and will be
    generalised for the final tool.
