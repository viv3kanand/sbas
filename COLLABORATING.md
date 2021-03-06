# Collaboration guidelines for this repo

## Tracking project progress

We track the progress of this project using issues. Feel free to add a new issue if it doesn't already exist.

## Extending functionality

As a collaborator of this project, to extend functionality to an existing script or notebook of this repo or add something new, please follow the `user-feature-branch  >> master` workflow described below.


 1. **Clone** this repo locally

 NOTE: If you already have this repo locally from a previous time, please make sure you have first done the following:
  - `git checkout master`, before you `checkout` in a new branch
  - `git pull`, to ensure you have absorbed the latest changes to master before you `git checkout -b <new-branch>`

 2. **Create a new feature branch**: From the latest master `git checkout -b my.github.user.name-<feature-description>` eg. `cristina-dependencies-figure1b`

 NOTE: We can infer from the branch name that `cristina` is working on adding `dependencies` for `figure1b`, so please keep the naming convention as described above.

 3. **Commit** changes to your own branch
 4. **Push** your work 
 5. Submit a **Pull request** to the **`dev`** branch **(dev <- my-feature-branch)** so that your team can review your changes. 
 6. Assign one of your team members as a `Reviewer`.

 NOTE: 

 Best practice is that pull requests are first merged into the `dev` branch. Subsequently the `dev` branch is merge into `master`. 
 Why? To perform some tests and also allow for better handling of conflicts when both you and  one of your other team members apply changes on the same file(s).

### The `templates` folder

To create a new Jupyter Notebook, copy the `template.ipynb` file located in the folder `templates` into the `jupyter` folder. Rename your file to be easy for your team to understand what figure of the publication this Notebooks addresses. Make sure the filename that you choose follows the convention `figure_id.ipynb` eg. `figure1c.ipynb`.

A typical workflow to create a new Notebook would look like this:

```bash
git clone https://github.com/TheJacksonLaboratory/sbas.git
cd sbas
git checkout my-branch

cp templates/template.ipynb jupyter/figure7g.ipynb
```

## Configuring environment (dependencies)

For detailed instructions on how to install the required dependencies please refer to the [dependencies/README.md](dependencies/README.md)

## Version control for .ipynb files

For humane and meaningful git diffs when working with `.ipynb` R kernel notebooks, we use `Jupytext` to convert from `.ipynb` to `.Rmd`. The automation of this step is still a work in progress so make sure to ask for advice from your collaborators when you are finished with editting an `.ipynb` file.


## The folder `data`

We use the folder **`data`** to store the artefacts that are generated during the analysis, so this folder should appear empty on GitHub. Large files cannot be pushed to GitHub as done with code. One different approach we can adopt for tracking medium sized artefacts up to 2GB. For this purpose we utilise the R package [`ropensci/piggyback`](https://github.com/ropensci/piggyback). This is still a work in progress and we will only sustain this for developing for as long as parts of this projects are a work in progress. See the following section about utilising cached intermediate files for more details.


## Using cached intermediate files 

Every notebook in this repo should be a self contained analysis. This means that all the required input files to execute the analysis are generated by the code or can be programmatically retrieved from archived objects. We maintain some archived objects attached in the GitHub Releases of the repository named [`lifebit-ai/lifebitCloudOSDREgtex`](https://github.com/lifebit-ai/lifebitCloudOSDREgtex). 

For decreasing the execution time of your notebook, especially in obtaining the `gtex` ExpressionSet by running  [`yarn::downloadGTEXv8()`](https://github.com/TheJacksonLaboratory/yarn/blob/a926b68bc9eca484bb003f57f99be057b707f8d3/R/downloadGTExV8.R) you can retrieve the file with the {piggyback} R package by executing the following command:

```r
# Download from archive if not available in ../data 
 if (!("gtex.rds" %in% list.files("../data/"))) { 
     message("Downloading GTEx v8 from GitHub Releases archive into the ../data/ directory ..\n") 
     piggyback::pb_download(file = "gtex.rds",  
                                 repo = "lifebit-ai/lifebitCloudOSDREgtex",  
                                 tag  = "fig1c_archive",  
                                 dest = "../data/") 
     message("Generating sha256sum for gtex.rds ..")     
     message(system("sha256sum ../data/gtex.rds", intern = TRUE)) 
     message("Done!\n")     
     message("Loading GTEx v8 rds object with readRDS from ../data/gtex.rds ..\n")     
     obj <- readRDS(file = "../data/gtex.rds" ) 
     message("Done!\n") 
 } 
```

## Archiving dependencies 

In the folder `metadata` we keep metadata about the artefacts and the environment where the latest analysis has been successfully completed. Please include the name of the figure that these dependencies refer to, by following the convention in the filename: 

#### `**"../metadata/" + figure_id + "_" + "*session_info.rds"**`

We keep the following metadata:

- the `utils::sessionInfo()` as an .rds file
- the `devtools::session_info()` as an .rds file

You can do so in R with the following snippet:


```r
### 2. Libraries metadata

figure_id <- "figure1b"

dev_session_info <- devtools::session_info()
utils_session_info <- utils::sessionInfo()

saveRDS(session_info, file = paste0("../metadata/", figure_id, "_",  "devtools_session_info.rds")
saveRDS(session_info, file = paste0("../metadata/", figure_id, "_", "utils_session_info.rds")
```

## Archiving file metadata
Similarly, we capture `sha256sum` hashes for the artefacts generated during the analysis, that are temporarily stored in the data folder before transfering to GitHub Releases via `{piggyback}`. To do so please write the `sha256sum` hashes in a .txt file.

- a `sha256sums.txt` with the `sha256sum` of each of the artefacts that our analysis has produced.

You can do so in R with the following snippet:


```r
### 1. Checksums with the sha256 algorithm

message("Generating sha256 checksums of the artefacts in the `..data/` directory .. ")
system("cd ../data/ && sha256sum * > ../metadata/figure_1b_sha256sums.txt", intern = TRUE)
message("Done!\n")
data.table::fread("../metadata/figure_1b_sha256sums.txt", header = FALSE, col.names = c("sha256sum", "file"))
```

It is advised that you append a sesction that captures metadata to the end of your R kernel `.ipynb` Notebook.
For an example have a look [here](https://github.com/TheJacksonLaboratory/lifebitCloudOSDRE/blob/e20eb44f4a9c341f8a3b08b71e007730fc8120d9/Rmd/Figure1cYarnVersion.Rmd#L472-L505).

As we will be working on more than one figures, make sure that you add in the filename the information about which figure the metadata refer to.
