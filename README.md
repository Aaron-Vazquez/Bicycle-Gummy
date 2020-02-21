<p align="center">
  <img src="https://user-images.githubusercontent.com/60892768/74993425-88d39900-5410-11ea-8643-b701551d0472.png">
</p>

# Bicycle-Gummy

The pipeline described was developed by Aarón Vázquez-Jiménez in the [Human Systems Biology Group](https://resendislab.github.io/) at INMEGEN.

## Atributtion
This pipeline was used to study functionally heterogeneity in the MCF7 cell line using Multicelluas Tumor Spheroids as tumor model. If you have any questions contact us at avazquezj(at)inmegen.gob.mx and oresendis (at) inmegen.gob.mx. 

## Getting Started

The complete pipeline starts with the fastq files. It ends with the files related to the differential gene expression analysis and the ones necceary to use the GSEA tool. The pipeline is formed by the following steps:

### Samples Processing

Raw files were processed througth ddSeeker pipeline ([Rogmanoli et al. 2018](https://link.springer.com/epdf/10.1186/s12864-018-5249-x?author_access_token=5GkMGb1JtnR887KgJLyh-m_BpE1tBhCbnbw3BuzI2ROCtPkWTeG4740r3l1fSwcikEVJgHZm9jmSpiShpuk2FV8ae_KUm1O2Kb8nf4xLIHEJfbwJ3tasCADHjVdZ23iWPECc69GpWSYTzS6lSpBaJA%3D%3D)) using **STAR** as aligner. As output, BAM files for every sample were created. Count matrix were generated by **DigitalExpression** function of the [Drop-Seq tools](https://github.com/broadinstitute/Drop-seq) with a value of NUM_CORE_BARCODES=5000.

A count matrix is generated for every of the 4 lines. All files can be merged into one.

#### Data Availabity 
The data used and discussed have been deposited in NCBI's Gene Expression Omnibus and are accessible through the accession number [GSE145633](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE145633).

### knee filter
Since we used droplet based tecnology for the hight throuput sequencing, data must be filtered by taking out droplets with no RNA from the biological samples, so, only informative cells remain. To implement knee filter we ploted the cumulaitve UMI sum   for all cells. The point where the firts inflection point occurs set the threshold.  

<p align="center">
  <img src="https://user-images.githubusercontent.com/60892768/74989579-6177ce80-5406-11ea-8e20-866bc38cdefd.png">
</p>

As outoput file, the count matrix of cells leftward of the threshold is saved. All samples are saved into one big matrix

### Gene Over-dispersion
Instead of using all genes into the analysis, we selected the most informative ones. To do so, firts taking hand of ![SCDE error models](https://hms-dbmi.github.io/scde/index.html) we got a matrix where overdispersion is computed of every gene and every sample. 
The basic concept is if a gene is affected by a process or condiiotn it will be reflected in their variance. A non affected gene will have the same value across data. An example is shown below.

<p align="center">
  <img src="https://user-images.githubusercontent.com/60892768/74991777-98e97980-540c-11ea-90ca-a90fe7ff38e1.png">
</p>

Overdispersion matrix for all data is saved in csv file

### Gene number selection
The number of genes used for further analysis has a relevant consiedration. If only the genes with greater variance (in the magnitude order of hundreds) gene expression analysis and pathway enrichment will give information about most significant proccesse. However, you will be discarting subtle proceeses with relevant information about the phenomena. Therefore, we propossed to discard the genes with a varition less than 15% of the total dynamic range.

<p align="center">
  <img width="450" height="450" src="https://user-images.githubusercontent.com/60892768/74992683-7193ac00-540e-11ea-948d-91af7130f1b1.png">
</p>

### Dimensionality Reduction
Since every data is in a multidimensionality space it is hard to associate similar characteristics among data. Therefore, a dimension reduction is a proper analysis to viasualize data properties in a bi-dimension space. Particularly for our data set, uMAP method works better due its objetive function definition (R pacakage **umap**). 

### Clustering
Instead of compared data according the sample labels, we mixed all data and performed clustering methods to re-group data by similarities in their gene expression profile. Two clustring methods were used to validate results and reduce possible unduced bias for each one of them.

Clustering was performed in two spaces. Firstly, in the uMAP bidimensional space. and secondly, in the multidimensional space of the distance of every cell giveng the variance. This was done to evaluate the anlaysis robustness and set possible considerations.

#### Kmeans
It is a supervised method based on distance of a centroid to the data, for kmeans implementation the number of cluster must be predefined. To select 

Kmeans method was implemented by R package **kmeans**, a multiple values for the number of groups was selectec, from 2 to 18. In each one, Sum of Squared estimate of Errors (SSE) was computed. So, the optimal clusters number was assigned when the elbow plot indicates the maximun change in SSE. This was done by computing the second derivative of SSE to find the optimal point.

<p align="center">
  <img width="450" height="450" src="https://user-images.githubusercontent.com/60892768/75059691-1eb80400-54a3-11ea-85b0-deb5973e2a44.png">
</p>

The projection of both inputs spaces clustered by kmeans are as follows. As can be seen, despite cluster asignation, uMAP input space has a more refined asignation. This consideration must be taken carefully, since multidimensional space clustering was not performed in a the uMAP space but it is plotted in the uMAP space.

<p align="center">
  <img width="900" height="450" src="https://user-images.githubusercontent.com/60892768/75060545-a2bebb80-54a4-11ea-9f04-5d9a5c28a42c.png">
</p>
