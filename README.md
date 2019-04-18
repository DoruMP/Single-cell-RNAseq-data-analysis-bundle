# Single cell RNAseq data analysis bundle

This repository contains a series of pipelines that can be used for processing, analysing and exploratory analysis of single cell RNA sequencing transcriptomics data using a server running on a Grid Engine. The scripts can also be used on personal machines and/or in interactive mode or by adapting the shel scripts to be used on other cluster softwares. 

These pipelines were developed for the study __Decoding the development of the blood and immune systems during human fetal liver haematopoiesis__ and due to their wider application they are being used and further extended for other projects.

The bundle includes:
* tools for building data sets from multiple CellRanger count tables
* data annotation aids
* training machine learning cell type classifiers
* doublet removal
* computing data reduction coordinates like tSNE, UMAP, force directed graph and diffusion map
* trajectory analysis
* interactive plots as html pages
* batch correction
* signature genes
* cell type comparison metrics and plots
* creating animated force directed graphs
* handling big data sets as Seurat objects
 
To create mode advanced data exploration apps and web portal(s) visit https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data. This repository will be referred in this documentation as [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data).

To run the pipelines you must download the entire bundle and transfer to a server/personal computer. The folder structure must be kept as it is.

## A proposed standard work flow (the order of steps is mandatory):

* create the seurat object (_seurat\_from\_count\_tables.sh_)
* train a doublet detector (_train\_doublets\_SVM.sh_)
* update doublet assignment in data (can use _Apply\_Classifier\_On\_Seurat\_Object(..)_ function in a customised script or, the easier approach is to run again the seurat object creation process with _seurat\_from\_count\_tables.sh_ and this time setting the argument _identify.doublets = TRUE_)
* subset the seurat object into singlets and doublets (_split\_seurat\_by\_category.sh_). It is advised to keep the doublets data (even if you have not further use for them) for future reference.
* compute dimensionality reduction coordinates (_add\_dr.sh_)
* make an annotation template and annotate (_make\_cell\_annotation\_template.sh_)
* optional: annotation can be made a lot easier by plotting annotation assisting word clouds (_wordclouds.sh_) and making an interactive heat map (interactive heat map tool at  [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data) _interactive\_heatmap\_dotplot_).
* important notice: it is highly advisable to use only alphanumeric characters in naming cell population. Some characters (e.g. "\\", "/") can create problems and raise errors with some pipelines. While many of these issues are solved for, it is still advisable as good practice to avoid fancy characters in naming. This is because it is imposible to predict all possible issues created by non-alphanumeric characters and even when they do trigger errors, the error messages are particularly vague in such situations. Here alpha-numeric is defined as the collection of Latin letters and Arabic digits.
* update the annotation (_update\_annotation.sh_)
* make all the apps that allow easy data exploration:
   * _plot\_dr.sh_ for interactive UMAP, FDG, tSNE and AGA; 
   * [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data) _interactive\_heatmap\_dotplot_ interactive heatmap but this time with the labels, not clusters; 
   * [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data) _web\_portal_ web portal tool for gene expression (you must have access to web server or alternatively set an Apache server on your local machine)
   * [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data) _gene\_grouping_ for gene expression patterns
   * _super\_markers.sh_ gets cell types signatures. You will understand the power of these signature when you input them in the interactive heat map (if you make one). This will be very useful for annotating new data, for supporting data annotation, for exploring expression patterns in new data sets or for designing new flow panels
   * optional: you could run again the word clouds because this also show DEGs as word clouds
   * optional: you could train a cell type classifier for fast integration of new data sets. However it is highly recommended that any published conclusions should be made on whole data annotation, not on classifiers results from this bundle (or any other type of machine learning classification/label propagations/projections made with tools that are not part of this bundle)
* it is recommended that you treat portals, doublet detectors, cell type classifiers and gene signatures as resources not as results. You should only share such resources with relevant people otherwise you might risk leaking results to others before publication.
* next steps are project specific

## Prerequisites

Python version 3.6
>pandas 0.22.0\
pptx 0.6.9\
patsy 0.5.0\
scanpy 1.2.2\
sklearn 0.19.1\
numpy 1.14.2\
scipy 1.0.0\
umap 0.2.3
 
R version 3.4.2
>Seurat 2.3.4\
dplyr 0.7.6\
reshape 0.8.8\
plyr 1.8.4\
ggplot2 3.0.0\
RColorBrewer 1.1.2\
BiocParallel 1.12.0\
gridExtra 2.3\
grid 3.4.2\
sva 3.26.0\
destiny 2.6.2\
ggplot2 3.0.0\
monocle 2.6.4\
harmony 0.1.0\
methods 3.4.2\
utils 3.4.2\
wordcloud 2.6
 
 ## Structure
 
 The main folder is called _single\_cell\_data\_analysis\_bundle_ and must contain:
 * data folder where all seurat object as kept as RDS file and scanpy objects as h5ad files
 * output folder where jobs save their output. This is where the user can get the results of running a pipeline
 * pipelines folder contain a folder for each pipeline
 * resources folder containing sample key, colour keys, cell type classifier, doublet detectors, options file etc.
 * tools folder - there is no reason why the user should be concern with this folder. It contains programs for running force-directed graph, AGA, UMAP, classifier prediction, doublet detection, pseudotime 3D viewer app builder and a file with lots of utilities. These are never required to be called directly by the user. 
 * the tools folder includes the file bunddle_utils.R. 
 * if you want to use the portal tools and fast gene expression explorer you must have an additional folder named portal_tool where these tools are stored and can be used as pipelines

## Colour keys

 Color keys compatible with the single cell analysis bundle can be generated using the interactive tool _color\_management.html_. Instructions can be found inside the interactive tool if opened in a browser (recommended: Chrome, Firefox; to avoid if possible: all versions of Internet Explorer and all versions of Microsoft Edge).

## Utilities

The functions in the _bunddle\_utils.R_ can be used in new pipelines or in customized scripts. If this is required check the parameter description for the required function and ensure access to required scripts that it need to call.

The _bunddle\_utils.R_:
* declares the tool.addr and python.addr variables. Change the python.addr if you want to use a different python version for your work but make sure first that the version you are trying to use has installed all the required packages.
* the functions _runFDG_, _RunUMAP_ are used to compute force-directed graph and UMAP coordinates for a seurat object. Although currently Seurat package has a function to compute UMAP which goes by the same name, the function in this bundle was created before Seurat published its umap computing function. Both the in-house and the Seurat RunUMAP functions do the same thing but because the bundle was build before Seurat had the ability to compute UMAP it is recomended to use the RunUMAP from the _bunddle\_utils.R_ script with the current bundle for compatibility reasons.
* __runFDG(pca.df, snn, iterations = 600, tool\_addr, python.addr)__
    * computes force directed coordinates on a seurat object. This function requires time and computation resources for big data sets.
    * @param `pca.df` the input data frame with variables as columns. In the pipeline this is used on the pca coordinates of a seurat object which can be retrieved at `seurat.obj@dr$pca@cell.embeddings`. The function is very flexible due to this input and can used on many types data formats not limited to a seurat object. Further flexibility of this function comes from the fact that other embeddings can be used besides pca (e.g. batch corrected pca stored at `seurat.obj@dr$harmony@cell.embeddings`).
    * @param `snn` shared nearest neighbor graph. In a seurat object this is available at `seurat.obj@snn`. If you want to use this function outside a pipeline you must make sure that the snn has been computed on the seurat object or if you are not using a seurat object you must computed using other available tools. You must pay attention to seurat object subsetting which as of writing this documentation does not recompute the snn.
    * @param `tool_addr` folder name were the bundle tools are stored. this function uses _force\_abstract\_graph/make\_fdg.py_ for the actual computations. If you need to use this function outside the pipelines you must make sure you have this script and set the tool.addr argument properly. 
    * @param `python.addr` python address. This is pre-set in all pipelines but having this as an argument allows the user to re-use the function in other scripts and choose the python version
* __RunUMAP(pca.df, tool\_addr, python.addr)__
    * computes umap coordinates. Currently Seurat packages has published a function with same name that computs UMAP coordinates on a seurat object. However the RunUMAP in this bundle is more flexible and can be used not just on a seurat object. This function is the fastest dimension reduction method (compared to PCA, tSNE and FDG).
    * @param `pca.df` the input data frame with variables as columns. In the pipeline this is used on the pca coordinates of a seurat object which can be retrieved at `seurat.obj@dr$pca@cell.embeddings`. The function is very flexible due to this input and can used on many types data formats not limited to a seurat object. Further flexibility of this function comes from the fact that other embeddings can be used besides pca (e.g. batch corrected pca stored at `seurat.obj@dr$harmony@cell.embeddings`).
    * @param `tool_addr` folder name were the bundle tools are stored. this function uses _umap/umap\_compute.py_ stored in the tools folder to compute the umap coordinates.
    * @param `python.addr` python address. This is pre-set in all pipelines but having this as an argument allows the user to re-use the function in other scripts and choose the python version
* __Apply\_Classifier\_On\_Seurat\_Object(seurat.obj, classifier.fname, tool\_addr, python.addr)__
    * this function applies an SVM classifier to the seurat objects and outputs predictions as a character vector;
    * the predictions can be added to the meta-data of the seurat objects
    * can be used to predict cell types or doublets. Make sure that the cell type classifier or doublet detector is relevant for the data you want to predict (e.g. do not use a thymus cell type classifier on spleen)
    * @param `seurat.obj` name of seurat object - data must be normalized before applying the function
    * @param `classifier.fname` folder name were the classifier is stored. cell type classifier are trained with the _train\_classifier.sh_ pipeline and doublet detectors are obtained with _train\_doublets\_classifier.sh pipeline_;
    * @param `tool_addr` folder name were the bundle tools are stored. This function calls the _predict\_by_classifier/predict.py_ script to load the classifier and must be present in the tool.addr folder. Inside the pipelines this argument is already set but if you want to use this function in other scripts you must set the proper tool.addr
    * @param `python.addr` python address. This is pre-set in all pipelines but having this as an argument allows the user to re-use the function in other scripts and choose the python version
* __make\_3D\_interactive\_page(data\_frame\_3D, tool\_addr, python.addr, save.to)__
    * outdated since the web portal was created (see [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data))
    * this function was used to create a 3D visualising html page. 
    * this function is the ancestor of the pseudotime web portal
    * this function is used by the _plot\_dr.sh_ pipeline to make a interactive tool for visualising diffusion map coordinates.
    * It can also used outside the pipeline to visualize other types of three-dimensional data sets (3D UMAP, 3D FDG, 3D tSNE) but this will be to be pre-computed
    * @param `data_frame_3D` data frame with the 3 dimensions in the first 3 columns, colours as hexdecimals in the forth columns and cell labels in 5th columns which must be named "Labels". The names of the other columns are not important. Make sue you do not have non-alphanumeric character in the labels (e.g. /, \, @ etc.) which can cause issues with the output.
    * @param `tool_addr` folder name were the bundle tools are stored. This function uses _interactive\_3D_viewer/html_WebGL\_3D\_viewer.py_ script which must be found in the tools folder. The function can be used outside the pipelines by setting the tool\_addr argument.
    * @param `python.addr` python address. This is pre-set in all pipelines but having this as an argument allows the user to re-use the function in other scripts and choose the python version
    * @param `save.to` address to save the resulted interactive page
* __make\_2D\_interactive\_page(data\_frame\_2D, tool\_addr, python.addr, save.to="./")__
    * this functions is similar to _make\_3D\_interactive\_page_ and it produces and interactive html page for vizualizing 2D data (UMAP, tSNE, FDG)
    * @param `data_frame_2D` has similar formating with the parameter data\_frame\_3D in function make\_3D\_interactive\_page the difference being that it features only two dimensions
    * @param `tool_addr` folder name were the bundle tools are stored. This function uses interactive\_2D\_viewer/html\_WebGL\_2D\_viewer.py script which be found in the tools folder.
    * @param `python.addr` python address. This is pre-set in all pipelines but having this as an argument allows the user to re-use the function in other scripts and choose the python version
    * @param `save.to` address to save the resulted interactive page
* __create\_gene\_expression\_viewer\_apps(seurat.obj, dim.type = 'umap', save.to, tool\_addr, python.addr, categories.colours=NA)__
    * deprecated
    * this has been replaced by the web portal creating tool (see [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data))
    * this can still be used by the pipeline _seurat\_to\_interactive\_gene\_expression.R_
    * this creates html interactive pages that allow data exploration for dimensional reduction plots and gene expression. However the data is embedded in the page so the results is heavy and will require time to load in a browser. It is not recommended to used the output on a web server. The output is useful for internal distribution of data. For other situation I recommend you use the web portal building tool.
* __plot.indexed.legend(label.vector, color.vector, ncols = 2, left.limit = 3.4, symbol.size = 8, text.size = 10, padH = 1, padV = 1, padRight = 0)__
    * function that creates a legend pdf file to be used with dimension reduction plots
    * this function is called in _plot\_dr.sh_
    * @param `label.vector` a character vector with the cell labels
    * @param `color.vector` a character vector with the colors for the labels written in hexdecimal format. hexdecimal colour keys can be created using the interactive tool _color\_management.html_
    * @param `ncols` number of columns to arrange the labels in the legend
    * @param `left.limit` left padding
    * @param `symbol.size` symbol size
    * @param `text.size` text size
    * @param `padH` horizontal padding
    * @param `padV` vertical padding
    * @param `padRight` right padding
* __dr.plot(point.labels, dr1, dr2, dr1.name, dr2.name, no.legend = F, plt.lb.sz = 5, txt.lb.size = 3, pt.size = .2, random\_state = 2, use.cols = NULL, use.labels = NULL, limits = NULL, annotate.plot = T, index.map = NA)__
    * function used to plot dimensionality reduced coordinates (e.g. tSNE, UMAP)
    * @param `point.labels` character vector of all labels in the data set
    * @param `dr1` numeric vector of first dimension coordinates
    * @param `dr2` numeric vector of second dimension coordinates
    * @param `no.legend` boolean indicating if to plot the legend inside the final plot. Sometimes it is desirable to plot the legend separatly to allow for more flexibility in arranging figure panels. In this case set this argument to FALSE and use plot.indexed.legend to create a separate legend figure
    * @param `plt.lb.sz` label size
    * @param `txt.lb.size` text label size
    * @param `pt.size` point size
    * @param `random_state` used for generating repeatable random colours when colours are not available
    * @param `use.cols` if null, then generate colours randomly
    * @param `use.labels` vector of labels
    * @param `limits` plot limits
    * @param `annotate.plot` boolean to indicate if plot should be annotated by with indices
    * @param `index.map` either a NA type or a numeric vector to set the indices of each label
 
## Running a pipeline

Each pipeline is run with the command `qsub name_of_pipeline.sh 'arguments inside single quotes'`
All pipelines must be run from their home directory
Inside the single quotes each argument must be take a value. These value should be in double quotes if strings or without otherwise (basically following R standard syntax);
A few pipelines do not take external arguments (_split\_seurat\_by\_category.sh_, _subset\_seurat.sh_ and _compute\_DEG.sh_). In these cases the arguments must be changed inside the R script.  
The standard assumptions of all pipelines are:
* all the data is in the data folder
* resource files are found in the resources folder
* any processed data resulted from the job will be saved to the data folder
* any other type of result will be saved in the output folder
* all pipelines (exceptions _split\_seurat\_by\_category.sh_, _merge\_seurat\_objects.sh_ and _subset\_seurat.sh_, _compute\_DEG.sh_, _gene\_discriminatory\_power\_analysis.sh_) when run create a dedicated folder inside the output folder. This carries the name of the pipeline + name of inputed data + unique time. Results other than processed data will be saved to this folder. This allows the user to map jobs with output. Inside the pipeline output folder there is a temporary folder created to stored transient processed data. This will be deleted most of the time when the job ends. In same case the transient processed data might be important for further work so the user should comment out the line `unlink(output_folder_material, ...)`

An example of running a pipeline:\
```qsub add_dr.sh 'seurat.addr = "fliv_lymphoid_Stage_1.RDS"; do.normalize = T; add.PCA = T; add.TSNE = T; add.UMAP = T; add.FDG = T; save.dr = F'```
* the above job will load the file "fliv\_lymphoid\_Stage\_1.RDS" from the data folder. If this file is not inside the data folder an error will occur and the job will be stoped;
* then the job will do data normalisation followed by the computation of PCA, tSNE, UMAP and FDG coordinates;
* lastly it will save the resulting Seurat object overwriting the file that was initially loaded (in this case "fliv\_lymphoid\_Stage\_1.RDS"). 
Jobs can be killed but take notice that if you terminate a process while it is writing to disk, the corresponding data will be lost.
For smaller data sets the scripts can be run on a local station. In such cases one cannot submit jobs using the `qsub` command. Instead the R and/or Python scripts can be run directly in interactive environments but the global parameters either pe passed directly to the script or set inside the scripts.

## Instructions for each pipeline


### integrate_new_pipeline/template.sh
 
 This is the template for building new pipelines compatible with the current bundle. It contains boiler plate code and can be used as the starting point for making new pipelines. It can handle any number of arguments, just place them in the arguments.list one on a line and ending with ".arg".

### seurat_from_count_tables.sh
* used to compile a seurat object from Cellranger output
* an example of runing this pipeline:\
`qsub seurat_from_count_tables.sh 'organ="liver"; ProjectName="Liver10x"; save.at="liver_all.RDS"; sequencing.types="normal"; annotate.cells = T; identify.doublets = T; cell.type.SVM = "classifier_svm_cell_type_ftlliv"; doublet.svm = "classifier_svm_doublets_ftlliv"'`
* reads the file _key.csv_ in the _resources_ folder. From this it selects the data based on organ name and sequencing type
* the seurat object is saved having the file name set by the argument _save.at_. The file will be saved in _data_ folder
* The arguments:
   * _organ_: string, name of the organ. Must exist in the _key.csv_ file (e.g. liver, thymus)
   * _ProjectName_: string, passed to the project argument when creating the seurat object
   * _save.at_: string, file name for the seurat object to be save to. File extension must be RDS. The file will be saved in the data folder
   * _sequencing.types_: strings, can be 'normal' for 3' data or 5GEX for 5' data
   * _annotate.cells_: boolean, use a trained SVM to automatically annotated the data. Make sure you have a trained SVM before asking for automatic annotation
   * _identify.doublets_: boolean, use a doublet detector to flag doublets in the dataset
   * _cell.type.SVM_: string, name of the cell type classifier. The classifier must exit in the resources folder and should have been created with the pipeline _train\_classifier_
   * _doublet.svm_: string, name of the doublet detector. The doublet detector must exist in the resource folder and should have been created with the pipeline _train\_doublets\_classifier_

### make_cell_annotation_template.sh
* makes an annotation template for data stored in a seurat object.
* it first clusters the data than computes DEGs and arranges the results in a form than can readily be used for data annotation
* the data file is overwritten at the end to include the new clustering.
* After clustering it is recommended to run the pipeline _interactive\_heatmap\_dotplot_ (see [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data)) on the clustered data because the interactive heatmap is highly useful for the annotation. But before doing that you should pre-append a string to the cluster index because pure cluster indices are not handled well in the interactive heat map (e.g. use Cluster\_110 instead of 110).
* an example of runing this pipeline:\
`qsub make_annotation_template.sh 'seurat.addr = "spleen_all.RDS"; clustering.res=30; DE.downsample=T'`
* The arguments:
   * _seurat.addr_: string, name of data file. Must be RDS format and contain a seurat object
   * _clustering.res_: integer, Louvain clustering resolution. Use higher values for bigger data sets. Always aim at over clustering for the purpose of data annotation. It is better to merge clusters that will be assigned the same labels than to have rarer population diluted inside bugger clusters and never being detected.
   * _DE.downsample_: boolean, set to TRUE for bigger data sets. This downsamples cell number in bigger clusters decreasing time significantly for DEG computation

### split_seurat_by_category.sh
* splits a seurat object in several subsets by the levels of a column in the meta data (e.g. splits by gender will create 2 smaller seurat objects, one for each gender)
* this pipeline does not use the output folder and the resulting data is saved in the pipeline home folder. I made this choice to allow investigating the resulting subsets of data before I transfer to the data folder to make sure I am overwriting any thing.
* this pipeline does not accept external arguments. Arguments must be changed inside the R script _split\_seurat\_by\_category.R_
* it is recommended that the results subsets are run through dimensionality reduction or batch correction before they are used for any downstream work. 
* an example of runing this pipeline:\
`qsub split_seurat_by_category.sh`
* The arguments:
   * _sort.by_ column meta data by which the data should be splitted
   * _seurat.addrs_ full or relative path for the RDS file storing the Seurat object
 
 ### merge_seurat_objects
* merges all the seurat object from a list fo file names
* an example of runing this pipeline:\
`qsub merge_seurat_objects.sh 'seurat.addrs = c("data1.RDS", "data2.RDS"); append_tag = T; tags_to_append = c("tag1", "tag2"); append_tags_at = "sample.ids"; save.at = "merged_data.RDS"'`
* the arguments:
    * _seurat.addrs_ character vector of RDS file names (must be at least 2) containing the seurat objects. Must include only the file name, not the full path. The assumption is that the datasets are found in the data folder inside the bundle
    * _append\_tag_ boolean flag to append a tag to the meta data to help keep track of the merged data sets. This has proved very useful for many downstream work so it is recommended to add the tags
    * _tags\_to\_append_ character vector containing the tags. Must be the same length as seurat.addrs. If _append\_tag_ is set to `FALSE` this argument will be ignored but should not be omitted from the list of arguments and can be set to `NULL` or `NA`
    * _append\_tags\_at_ meta data column where to append to tags
    * _save.at_ RDS file name where to save the merged seurat object. Must contain only the file name, not the path. It will be save to the data folder. Make sure the file names does not exist before running the pipeline to avoid over-writing of data.

### subset_seurat.sh
* used to subset a part of the data set and save to a new RDS file as a seurat object
* this pipeline does not use the _output_ folder
* this pipeline does not take external arguments. Arguments must be written inside the R script _subset\_seurat.R_
* an example of runing this pipeline:\
`qsub subset_seurat.sh`
* the arguments:
    * _seurat.obj.addr_ full or relative path of the input RDS file 
    * _save.at_ full or relative path of the output RDS file 
    * _process_ boolean flag to run common data processing (normalisation, scaling, variable genes computation, PCA). It is recommended to set this to `TRUE`. However there are cases when data processing might not be required so in this case time can be saved by setting this argument to `FALSE`.
    * _add.dr_ boolean flag to compute tSNE, UMAP and FDG. It is recommended to set this to `TRUE`. If `TRUE` the previous argument also be set to `TRUE` otherwise an error will raised. There are times when these computations are not required so set this argument to `FALSE`. Not processing and not adding dimensionally reduction also ensures light-weight data sets which are easy to transfer over the web. 

### add_dr.sh
* computes tSNE, UMAP and FDG on a seurat object
* also includes a script called _add\_dr\_COMBAT.R_ which can be used to run batch correction using COMBAT implemented in Python. However this is very slow on data sets with more than 50k cells. COMBAT correction changes gene expression
* the default of this pipeline is not to use COMBAT batch correction. If this is required edit the _add\_dr.sh_ file by replacing _add\_dr.R_ with _add\_dr\_COMBAT.R_
* an example of runing this pipeline:\
`qsub add_dr.sh 'seurat.addr = "data_scseq.RDS"; do.normalize = T; add.PCA = T; add.TSNE = T; add.UMAP = T; add.FDG = T; save.dr = F'`
* the arguments are:
    * _seurat.addr_ file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in _data_ folder
    * _do.normalize_ boolean to normalize data. This must be set to `TRUE` if the input data has not yet been normalized. If this is set to `FALSE` but the data has not been previously normalised and error will occur and the job will be killed
    * _add.PCA_ boolean to compute PCA. Same principles and warnings as for the previous argument
    * _add.TSNE_ boolean to compute tSNE
    * _add.UMAP_ boolean to compute UMAP
    * _add.FDG_ boolean to compute FDG
    * _save.dr_ boolean to save the dimensionality reduction coordinates as a data frame in a csv file in the pipeline output folder. This is particularly useful for bigger data sets which either take long time to load or are not manageable on personal computers at all. In those case having the coordinates and meta data in csv files will save time

### compute_DEG.sh
* computes differential expressed genes (DEGs) on a seurat object
* this is different from _make\_cell\_annotation\_template.sh_ which computes DEGs only on clusters
* it allows computation of DEGs on any meta data category and also can be used post-annotation to get cell type DEGs
* this pipeline does not take external arguments. Arguments must be set inside the R script _compute\_DEG.R_
* an example of runing this pipeline:\
`qsub compute_DEG.sh`
* the arguments:
    * _seurat.addrs_ full or relative path of the RDS file containing the Seurat object. 
    * _save.to file_ name where to save markers genes in csv format
    * _DE.downsample_ boolean to downsample data if to big. Set this to `TRUE` for big data sets.
    * _category_ meta data column by which DEGs are computed (e.g. cell.labels, stages)

### violin_plots.sh
* makes violin plots using set genes from a seurat object
* Seurat package already has ability to construct violin plots. However this pipeline is advantageous for big data sets that are difficult or impossible to manage on personal computers or interactive sessions
* an example of runing this pipeline:\
`qsub violin_plots.sh 'seurat.addr = "data_scseq.RDS"; set.ident = "cell.annotations"' type.to.colours = "data_color_key.csv"; cell.labels = "fname_with_labels"; plot.width = 10; plot.height = 10; features.file = "fname_with_genenames"'`
* the arguments:
    * _seurat.addr_ file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in _data_ folder.
    * _set.ident_ meta data column to set the identity of cells
    * _type.to.colours_ name of csv file that contains the colour key (mapping between categories in the set.ident and colours). Must contain only the file name not the full or relative path because the assumption is that this is a resource file that is placed in _resource_ folder. Color keys compatible with the single cell analysis bundle can be generated using the interactive tool _color\_management.html_
    * _cell.labels_ indicates what categories to include in the violin plot. If set to `"all"` it will use all the categories. If a subset of categories is desired you must pass the file name that exits in the resource folder and contain one category name per line
    * _plot.width_ numeric plot width
    * _plot.height_ numeric plot height
    * _features.file_ name of file that must be found in the resource folder and to contain a gene name per line
* and example of _cell.labels_ file:
```
MEP_kidney
Mast cell_kidney
Megakaryocyte_kidney
Late Erythroid_skin
Mid Erythroid_skin
MEP_skin
Megakaryocyte_skin
Mast cell_skin
```
* an example of _features.file_ file:
```
UNG
MSH6
CD19
TNFRSF13C
TRNT1
PIK3CD
MOGS
INO80
TNFRSF13B
TNFSF12
```

### gene_heatmap_and_spotplot.sh
* plots a heat map and a dot plot using selected gene names and cell types from a seurat object
* an example of runing this pipeline:\
`qsub gene_heatmap_and_spotplot.sh 'seurat.addr = "data_scseq.RDS"; set.ident = "cell.labels"; genes.to.plot = "fname_genes"; cell.types = "fname_with_labels"; cluster.genes = T; diagonalize = F; plot.dims = c(10, 10, 10, 10); save.gene.order = T;'`
* the arguments:
    * _seurat.addr_ file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in _data_ folder.
    * _set.ident_ meta data column to set the identity of cells
    * _genes.to.plot_ name of file that must be found in _resource_ folder and to contain one gene name per line
    * _cell.types_ indicate what categories to include in the violin plot. If set to `"all"` it will use all the categories. If a subset of categories is desired you must pass the file name that exits in _resource_ folder and contain one category name per line
    * _cluster.gene_ boolean if cluster genes
    * _diagonalize_ boolean if diagnolize genes (i.e. placing highest values in each row closer to the diagonal to make for better vizualisation)
    * _plot.dims_ numeric vector containing the widths and heights of the heat map and dot plot respectively
    * _save.gene.order_ this is useful if the ordering of genes from diagonalization and/or clustering has created good visualisation and the user needs to store the ordered genes for future plots using the same gene set
* an example of _genes.to.plot_ file:
```
TRAV2
TRAV3
TRAV4
TRAV5
TRAV6
TRAV7
```

### seurat_to_interactive_gene_expression.sh
* before considering using this you should know that the web portal tool might be a better option (see [__Fast Portals__](https://github.com/DoruMP/Fast-data-portals-for-scRNAseq-data))
* this pipeline creates an interactive app that allows data exploration in a browser. Part of its limitations is that it shows only the variable genes and that expression data is embedded in the interactive page making it heavy and requiring time to load in a browser. The web portal tool has a further advantage in that it can build portals from both Seurat and Scanpy objects. However if all you need is to share a data exploration app with your team members or collaborators without disclosing data before publication this might be the pipeline for you
* an example of runing this pipeline:\
`qsub seurat_to_interactive_gene_expression.sh 'seurat.addr = "data_scseq.RDS"; dim.type = "umap";'`
* the arguments:
    * _seurat.addr_ file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in _data_ folder.
    * _dim.type_ dimensionality reduction type to be used in the resulted app (e.g. tsne, umap, fdg). Must exist in the `@dr` slot of the seurat objects, otherwise it will raise error and kill the process. You can use the _add\_dr.sh_ pipeline prior to this if you are not sure

### plot_dr.sh
* pipeline used to plot all dimensionally coordinates and colour the points by any column in the meta data
* additional it can also compute diffusion map and AGA graph
* warning about diffusion maps: most of them will make no sense if the data does not contain a true lineage. To ensure the diffusion map will make sense care must be taken in up stream work flow and must ensure removal of doublets, removal of outliers if possible and most importantly that the cell types in the data are part of a true biological lineage and only one lineage
* warning about AGA: some times AGA results are difficult to interpret especially when running on data sets that contain too many cell types. Establishing what too many means is up to trial and error and experience
* this pipeline also creates interactive apps (as html pages) that allow exploration of the dimensionality reduction coordinates and AGA structure. Furthermore diffusion maps can be visualised in a 3D interactive enviroment
* The 2D interactive app are build for tSNE, UMAP, FDG but take notice that these should be computed prior to running this pipeline (see _add\_dr.sh_ or _batch\_correction.sh_).
* The AGA interactive app includes instructions when opened on a browser. See general description of single cell analysis pipeline for browser compatibility.
* the arguments:
    * _seurat.addr_ file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in _data_ folder.
    * _plot.by_ indicates the meta data column(s) to be used in colouring the plots. Can be one string if only one column is used or a character vector if more columns are required
    * _type.to.colours_ indicates colours for all categories for all meta data columns chosen to plot. Can be one string if only on column is chosen or a character vector if more columns are chosen. Each value can be `NA` if random colours are required or a color key file in csv format found in _resource_ folder. See the _color\_management.html_ tool for generating color keys compatible with the single cell analysis bundle
    * _runDiffusionMap_ boolean to run diffusion map. Set this to `TRUE` only after considering the warnings about diffusion maps.
    * _runAGA_ boolean to run AGA

### gene_discriminatory_power_analysis.sh
  - this pipeline trains a random forest for classifying cell labels in a seurat object using a set of gene names
  - it was created to assess discriminatory power of gene sets using a random forest classification report
  - the random forest was chosen for this purpose due the partial similarities between it classifying mechanism and flow sorting
  - this is a not a pipeline for training classifiers. if that's what you need check train_classifier.sh pipeline
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - feature_genes name of file that must be found in the resource folder and to contain one gene name per line
    - cell.types name of file found in resource folder and contains one cell type per line
    - save.to.dir name of folder to save results
    - ident.set name of column in meta data to used for partitioning the data
    - type.to.colours name of csv file found in the resource folder that contains the cell type to colour mapping. To generate colour key compatibly with the single cell analysis bundle check the interactive tool color_management.html

### pseudotime.sh
  - used for trajectory analysis
  - uses diffusion map. please check plot_dr.sh pipeline for warnings about diffusion maps
  - computes pseudotime and (optionaly) genes that vary with pseudotime
  - it outputs plots of normalized and non normalized gene expression by pseudotime
  - it also produces an interactive page used for visualising some of the top variable genes. Check the pseudotime portal creation tool for a better alternative richer in functionalities and showing a higher number of genes.
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - ident.set name of column in meta data to used for partitioning the data
    - cell.types character vector containing a list of cell types to be used in the trajectory. You must replace every white space in the names ("") with double sign ("@@"). Check commands examples for reference. This the most important parameter for this pipeline - check the warnings about diffusion map.
    - root_cell_type name of root of trajectory. must exist in cell.types. Must have the same formating (replacing " " with "@@")
    - var.genes set this to NA to flag the computation of all variable genes. If instead f computing variable genes one needs to analyse expression pattern of certain genes this argument must be the name of file placed in resource folder and containing a gene name per line.
    - type.to.colours name of csv file found in the resource folder that contains the cell type to colour mapping. To generate colour key compatibly with the single cell analysis bundle check the interactive tool color_management.html

### fdg_animation_write_input
  - this pipeline is used for the fist step in creating an animated force direct graph. 
  - the pipeline name is write_input.sh
  - this saves important material to be used in the animation creation (a legend figure, a csv file mapping cell types to colours, PCA data and the shared nearest neighbour graph in sparse matrix format)
  - this pipeline does not take external arguments
  - the entire workflow for animation creation continues with the tools in the folder force_abstract_graph_2Danimation. It is recomended to run this in a supervised way (on a personal computer not on a server). Inside this folder there must be an empty folder called input. This where the material created by write_input.sh must be transfered. Then start make_fdg_animation.py. Inside this script there is the line "subprocess.call(["Rscript", "make_plots.R"], shell = True)". This might fail on different platform. The solution in this case is run the script make_plots.R independently then carry on with make_fdg_animation.py from where you left.
  - the tools in force_abstract_graph_2Danimation need installation of an in-house modified version of fa2 package. The modified version can be found at force_abstract_graph_2Danimation/iterative_fa2/. To install this go to force_abstract_graph_2Danimation/iterative_fa2/ and run "python3.6 setup.py install" (change according to your python version)
  - you must also have open computer vision python package opencv2 pre-installed. Check the online documentation on how to install this on your platform
  - the write_input.sh has only 2 arguments:
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - cell.type.to.colour name of csv file found in the resource folder that contains the cell type to colour mapping. To generate colour key compatibly with the single cell analysis bundle check the interactive tool color_management.html

### train_classifier.sh
  - used for training cell type classifiers
  - classifier type SVM using PCA input. The classifier is saved as 3 files: SVM (model.pickle), PCA projection (pca.pickle), and a list of feature genes (feature_genes.RDS) 
  - it also saves classification reports (classification_report.txt, confusion_matrix.csv and confusion_matrix.pdf). This file could be important in gauging the utility of the classifier
  - the classifier files and report are saved in a user-defined folder placed in the resource folder
  - classifiers can be applied using the function Apply_Classifier_On_Seurat_Object (see the bunddle_utils.R script)
  - IMPORTANT NOTICE: classifiers must be used only on the same type of data that it was trained on e.g. a classifier for cells in liver should only be applied on liver data. If for example you will apply the liver cell types classifier on thymus the classifier will only "see" the cell types it was trained to see and your results will be wrong. Furthermore most of the time the use of classifiers in a cross-tissue manner is highly unprofessional and indicate severe incompetence and a potential need for staff replacement. There are however a few (very few) exceptions where a classifier could be used on a different tissue from its training (e.g. the origin of the unlabelled data is not known; used as a pseudo-metric for cross-tissue cell type similarities).
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - marker.genes name of csv file storing DEGs by cell population. The top 20 DEGs for each cell type are used as feature genes. The file must be found in the folder resource/marker_genes. See compute_DEG.sh on how to obtain DEGs for a data set. 
    - save.at folder name were classifier files and report are saved. The folder will be created in the resource folder because classifiers are consider resources.
    - classifier name of classifier script. Currently the SVM is the only one implemented so this argument should always be set to "svm". However this argument allows extending of this pipeline to work with other classifiers. During development other classifiers were logistic regression, random forest, ada boost and multi-layer perceptron. However the SVM showed the best accuracy and recall over all so only the svm was kept. 

### train_doublets_SVM.sh
  - pipeline for training doublet detector
  - each doublet detector should only be used on the data set it was trained. While this sounds conter-intuitive for the machine learning users, all methods for doublet detection use this approach. The detector is actually trained on original data merged with dummy doublet data but it is applied only on real data. The idea behind ML-based doublet detectors is that a trained ML will find the optimum separation plane between dummy doublet and real data and because overfitting is avoided by regularisation, the plane will also separate a big part of the real doublets. It is intrinsically difficult to validate the identified doublets but the doublet removed by the the detectors created by this pipeline a) have a higher UMI and gene number counts b) have detected no doublet in plates data at least so far c) has improved downstream analysis, especially diffusion maps and UMAPs.
  - It is usually better to first train a doublet detector, then remove doublets and only then do annotation (manual with the annotation template or automatically with trained classifiers)
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - save.to folder name were classifier files and report are saved. The folder will be created in the resource folder because doublet detectors are consider resources.

### pull_data_subset_from_seurat.sh
  - used for saving to disk parts of a data set. This is especially for bigger data sets or when needing to run statistics for specific question that do not require the entire data and allows work in an interactive session
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - genes.file name of file found in the resource folder that list required genes to subset having one gene name per line. If this is NA than all genes will used
    - set.ident name of column in meta data to used for partitioning the data
    - cell.types name of file found in the resource folder that lists the cell type for partitioning the data with one cell type per lane. If set to "all" all the cell types will be used.
    - save.meta boolean to save meta data. Having the meta data in separate file it is very useful when working with big data sets and will save a lot of time
    - save.raw.data boolean to save the raw counts data
    - save.data boolean to save normalized gene expression data
    - save.dr boolean to save all dimensionally reduction coordinates

### clustering_comparison.sh
  - compares Louvain clustering with 2 other types of clustering (agglomerative clustering and Gaussian mixture)
  - metrics are rand index and adjusted mutual information
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - set.ident name of column in meta data to used for partitioning the data
    - n_clusters number of clusters
    - type.to.colour name of csv file that contains the colour key (mapping between categories in the set.ident and colours). Must contain only the file name not the full or relative path because the assumption is that this is a resource file that is placed in the resource folder. Color keys compatible with the single cell analysis bundle can be generated using the interactive tool color_management.html

### batch_correction.sh
  - perform batch correction at the level of principal components (also called data integration) using harmony R package
  - the dimensionality reduction coordinates are computed based on harmony principal components
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - correct.by meta data column to be correct by. Usually this should indicate sample assignment
    - save.at name of file to save the batch corrected data as Seurat object in RDS format. The file will be saved in the folder data
 
### wordclouds.sh
  - creates word clouds of cell types and DEGs for each cell population or other categories in the data set
  - cell type word clouds are generated based on tag mentions weighted by gene expression
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - set.ident name of column in meta data to used for partitioning the data

### update_annotation.sh
  - used to update the annotations in a seurat object following manual annotation
  - annotation should be kept in csv file. The empty template is produced by the make_cell_annotation_template.sh which requires only filling in the cluster assignment
    - seurat.addr file name of the RDS object containing the input data as a seurat object. Must include only the file name not the path because the assumption is that data files are kept in the data folder.
    - make.app boolean to make interactive page for visualising the annotated data. Usually not necessary as better alternatives have been made available since this pipelines was made (check plot_dr.sh and the web portal tool). Make sure dimensionality reduction has be computed before setting this argument to TRUE.

### cell_comparison.sh
  - make 1-to-1 comparison between cell types with the same data sets or between 2 data sets
  - comparisons include correlation plots, AGA score plots and DEGs for each comparison
    - seurat.query.addr query data (same format as the argument seurat.addr in other pipelines)
    - seurat.ref.addr reference data (same format as the argument seurat.addr in other pipelines). This can have the same value as seurat.query.addr if the reference cell types and query cell types come from the same data set
    - set.ident.query query set identification meta data column (same format as the argument set.ident in other pipeliens)
    - set.ident.ref reference set identification meta data columns (same format as the argument set.ident in other pipeliens)
    - cell.types.query query cell types (same format as cell.types in other pipelines)
    - cell.types.ref reference cell types (same format as cell.types in other pipelines)
    - dims.plot width and height in inches for the correlation and AGA plots
    - compute.DEGs boolean to compute DEG and plot them as jitter plots

### super_markers.sh
  - selects and order DEGs most specific for each cell population in the data set
  - the result are cell type signatures i.e. list of genes highly relevant for cell annotations
  - this script has only one argument to set: seurat.addr which follot the same format as in other pipelines
  - set.ident is set to "cell.labels". Change this to what is relevant for your project if: either you are not interested in cell type signatures or the cell annotations are not kept in the "cell.labels" meta data column (this might be the case if you have not started data analysis with this bundle)
  
  