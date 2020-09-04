# PHLASH

*STILL IN DEVELOPMENT.*

## Introduction

*Phlash* is an interactive web application that automates the manual process of bacteriophage genome annotation. Please watch this [demo](https://www.youtube.com/watch?v=dxf7szHk5aI&feature=youtu.be) to get a preview of how it works! 

## Background

Bacteriophages are diverse, ubiquitous organisms that create the microbial driven world we currently live in. Hundreds of students and faculty members at universities across the nation participate in the [SEA-PHAGES](https://seaphages.org/) program to discover and analyze uncharted bacteria, causing phage genomes to be added to GenBank at an exponential rate. 

In order to annotate genomes, researchers currently use bioinformatic annotation tools—[GeneMark](https://www.ncbi.nlm.nih.gov/pubmed/15980510) and [DNA Master](https://phagesdb.org/DNAMaster/)—to identify at least 80% of genes correctly. While these tools are a vast improvement from past approaches to genome annotation, each software still comes with its own limitations, including access and usability. Even after auto-annotation, researchers are required to manually reexamine the phage’s entire genome and add, modify, or delete genes depending on the data provided from the aforementioned tools. Such a task is not always easy and predicated on varying principles, including manual inspection between possible start sites, relationship to the closest upstream gene, highest coding potential, longer possible frame, etc. Although bacteriophage genomes are relatively small compared to eukaryotic genomes, this process is manually intensive and can consume hours of research that can otherwise be used more productively. 

Evidently, this tedious nature of genome annotation is a major bottleneck for extensive analysis of phage genomes and their implications, yet it is also a critical step to advancing research. Once researchers complete genome annotation, they perform, for example, [comparative analysis](https://www.ncbi.nlm.nih.gov/books/NBK20253/) of phage genomes, which reveals common architectural themes in genomes, unique protein functions, and genetic diversity. Such information furthers existing biological knowledge about the phage’s phamilies, as well as its relationships to other organisms, namely humans. The results of current phage genome annotations have extensively contributed to a variety of [phage-based therapies](https://en.wikipedia.org/wiki/Phage_therapy) in medicine and agriculture, especially regarding drug-resistant and long-term bacterial infections. Additionally, the human body’s abundance of phages and other viruses is recognized as an important mediator of human health and well-being, therefore emphasizing the need to continue increasing knowledge about phage communities. 

We developed *Phlash* to allow phage genome annotation to be as quick and efficient as possible. *Phlash* reduces the manual challenges by combining functionalities of GeneMark and DNA Master and outputting submission-ready GenBank files. By dramatically accelerating the pace of bacteriophage genome annotation, we hope that this application will enable phage hunters will then be able to spend less time annotating and more time contributing to our knowledge on the genetic diversity of bacteriophages and their implications. 

## User Workflow

1. Upload a FASTA file and specified output files from GeneMark and DNA Master.
2. Observe data that has been uploaded. Add, update, or delete gene calls as is approrpriate.
3. Wait as *Phlash* compares gene calls from both tools and determines a status for each prediction. 
    - Pass
    - Fail
    - Need more information
4. If passed, leave gene prediction as is.
5. If failed, determine the best possible start site based on the evidence provided.
6. If more information is needed, evaluate the evidence provided and make an informed decision to leave, update, or delete the gene call.
7. For each gene call, run a BLASTp search to assign a potential gene function.
8. Download a submission-ready GenBank file. 
