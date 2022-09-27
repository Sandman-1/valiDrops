# valiDrops

valiDrops is an R package for detecting high-quality barcodes in droplet-based single-cell or single-nucleus RNA-seq datasets. In addition, valiDrops is capable of detecting and flagging apoptotic cells.

# Installation

valiDrops can be installed directly from GitHub using either {[devtools](https://cran.r-project.org/web/packages/devtools/index.html)} or {[remotes](https://cran.r-project.org/web/packages/remotes/index.html)}. In addition to valiDrops, it is high advisable to install {[presto](https://github.com/immunogenomics/presto)}, which yields a 80 - 100X speed-up when using valiDrops. Both packages can be installed using the following commands:

```{r}
install.packages("remotes")
remotes::install_github("madsen-lab/valiDrops")
remotes::install_github("immunogenomics/presto")
```

# Usage

valiDrops uses the raw count matrix as input and produces a series of plots as well as a data.frame containing various quality metrics, which can be used to select barcodes for further analysis. Here, we demonstrate how to load data processed using either [STARsolo](https://github.com/alexdobin/STAR) or [CellRanger](https://support.10xgenomics.com/single-cell-gene-expression/software/overview/welcome), but any preprocessing tools that produce unfiltered count matrices are applicable. For example, data processed with [alevin-fry](https://github.com/COMBINE-lab/alevin-fry) can be imported using {[fishpond](https://bioconductor.org/packages/release/bioc/html/fishpond.html)}. Next, we show how to run valiDrops on the imported data, and create a {[Seurat](https://cran.r-project.org/web/packages/Seurat/index.html)} object for further processing. For getting started with further processing we refer users to the [Seurat vignettes](https://satijalab.org/seurat/). 

```{r}
## Load libraries
library(Matrix)
library(Seurat)
library(valiDrops)

## Loading data
# Load STARsolo data
data <- Matrix::readMM("path_to_data/raw/matrix.mtx")
barcodes <- read.delim("path_to_data/raw/barcodes.tsv", header=FALSE)
features <- read.delim("path_to_data/raw/features.tsv", header=FALSE)
colnames(data) <- barcodes[,1]
rownames(data) <- features[,1]

# Load CellRanger data
data <- Matrix::readMM("path_to_data/outs/raw_feature_bc_matrix/matrix.mtx.gz")
barcodes <- read.delim("path_to_data/outs/raw_feature_bc_matrix/barcodes.tsv.gz", header=FALSE)
features <- read.delim("path_to_data/outs/raw_feature_bc_matrix/features.tsv.gz", header=FALSE)
colnames(data) <- barcodes[,1]
rownames(data) <- features[,1]

## Run valiDrops
valid <- valiDrops(data)

## Setup data
valid.subset <- valid[ valid$qc.pass == "pass",]
rownames(valid.subset) <- valid.subset[,1]
valid.subset <- valid.subset[, grep("fraction", colnames(valid.subset))]
data.subset <- data[,colnames(data) %in% rownames(valid.subset)]
data.subset <- data.subset[,match(rownames(valid.subset), colnames(data.subset))]

## Create a Seurat object
seu <- CreateSeuratObject(data.subset, project = "valiDrops", meta.data = valid.subset, min.cells = 1, min.features = 1)
```

# Detecting apoptotic cells

valiDrops can detect apoptotic cells by providing a flag to the valiDrops() function or by running the label_apoptotic() function seperately. Here, we demonstrate how to label apoptotic cells are part of the valiDrops workflow assuming you have already loaded libraries and imported your datasets.

```{r}
## Run valiDrops
valid <- valiDrops(data, label_apoptotic = TRUE)

## Remove apoptotic cells and create a Seurat object
# Setup data
valid.subset <- valid[ valid$qc.pass == "pass" & valid$apoptotic == FALSE,]
rownames(valid.subset) <- valid.subset[,1]
valid.subset <- valid.subset[, grep("fraction", colnames(valid.subset))]
data.subset <- data[,colnames(data) %in% rownames(valid.subset)]
data.subset <- data.subset[,match(rownames(valid.subset), colnames(data.subset))]

# Create a Seurat object
seu <- CreateSeuratObject(data.subset, project = "valiDrops", meta.data = valid.subset, min.cells = 1, min.features = 1)

## Keep apoptotic cells, and create a Seurat object indicating which barcodes are likely to be apoptotic cells
# Setup data
valid.subset <- valid[ valid$qc.pass == "pass",]
rownames(valid.subset) <- valid.subset[,1]
valid.subset <- valid.subset[, grep("fraction", colnames(valid.subset))]
data.subset <- data[,colnames(data) %in% rownames(valid.subset)]
data.subset <- data.subset[,match(rownames(valid.subset), colnames(data.subset))]

# Create a Seurat object
seu <- CreateSeuratObject(data.subset, project = "valiDrops", meta.data = valid.subset, min.cells = 1, min.features = 1)
```

