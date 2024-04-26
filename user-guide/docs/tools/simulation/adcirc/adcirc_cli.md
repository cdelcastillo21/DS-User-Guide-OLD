
#### ``adcirc`` Command Line Options

| Option                            | Description                                                                                   | Special Notes                                                                                 |
| --------------------------------- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `-I INPUTDIR`                     | Set the directory for input files.                                                            |                                                                                               |
| `-O GLOBALDIR`                    | Set the directory for fulldomain output files.                                                |                                                                                               |
| `-L`                              | Write subdomain output to PE* directories in parallel mode.                                   | Does not affect NetCDF hotstart files.                                                        |
| `-S`                              | Write subdomain binary hotstart files to PE* directories in parallel mode.                    | Does not affect NetCDF hotstart files.                                                        |
| `-R`                              | Read subdomain binary hotstart files in parallel mode.                                        | Does not affect NetCDF hotstart files.                                                        |
| `-W NUM_WRITERS`                  | Dedicate NUM_WRITERS MPI processes to writing ascii output files.                             | Affects ascii formatted fort.63, fort.64, fort.73, and fort.74 files.                         |
| `-Ws NUM_SPLIT_WRITERS`           | Similar to `-W`, but creates a new file at each data write.                                   | Affects ascii formatted output files, creating a new file for each data write.                |
| `-H NUM_HOTSTART_WRITERS`         | Dedicate processes to writing binary hotstart files, for 2DDI runs without harmonic analysis. | Does not affect NetCDF hotstart files. Suitable only for 2DDI runs without harmonic analysis. |
| `-M`                              | Write subdomain harmonic analysis files in parallel mode.                                     | Useful for tidal analyses with many constituents.                                             |
| `-Mft MESH_FILE_TYPE`             | Specify the mesh file type (ascii or xdmf).                                                   | Introduced in ADCIRC v51.                                                                     |
| `-Mfn MESH_FILE_NAME`             | Specify the full domain mesh file name.                                                       |                                                                                               |
| `-Nft NODAL_ATTRIBUTES_FILE_TYPE` | Specify the nodal attributes file type (ascii or xdmf).                                       | Introduced in ADCIRC v51.                                                                     |
| `-Nfn NODAL_ATTRIBUTES_FILE_NAME` | Specify the full domain nodal attributes file name.                                           |                                                                                               |

#### ```adcprep``` Command Line Options

| Option                | Description                                                                                   | Special Notes                                                                    |
| --------------------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `--np NUM_SUBDOMAINS` | Decompose the domain into NUM_SUBDOMAINS subdomains.                                          | Required for parallel computation.                                               |
| `--partmesh`          | Partition the mesh only, resulting in a partmesh.txt file.                                    | Should be done first. Generates partmesh.txt for subdomain assignments.          |
| `--prepall`           | Decompose all ADCIRC input files using the partmesh.txt file.                                 | Requires previous execution with `--partmesh`. Expects default input file names. |
| `--prep13`            | Only redecompose the nodal attributes (fort.13) input file.                                   | Requires that `--prepall` has already been performed.                            |
| `--prep15`            | Only redecompose the control model parameter and periodic boundary conditions file (fort.15). | Requires that `--prepall` has already been performed.                            |
| `--prep20`            | Only redecompose the non-periodic, normal flux boundary condition file (fort.20).             | Requires that `--prepall` has already been performed.                            |
| `--prep88`<br>        | Only redecompose the upland river initialization file (fort.88).                              | Requires that `--prepall` has already been performed.                            |

**``adcprep`` Runs**

The usual workflow of running `adcprep` consists of two steps - (1) partitioning of the mesh into sub-domains that each core will work on. (2) Decomposing other input files over the partitioned mesh.

Note that running `adcprep` alone with no command line options will bring up an interactive menu. 

Common `adcprep` options used include:

- **Partitioning Mesh Only**

  ```
  adcprep --partmesh --np 32
  ```
  
  This command partitions the mesh into 32 subdomains, creating a partmesh.txt file.

- **Preparing All Input Files**

  ```
  adcprep --prepall --np 32
  ```
  
  Utilizes the previously created partmesh.txt file to decompose all input files into PE* subdirectories.

### PADIRC Runs

Some common options used when using PADCIRC are the following:

- **Specifying Input/Output Directories**

  ```
  padcirc -I /path/to/input -O /path/to/output
  ```
  
  Looks for input files in `/path/to/input` and writes output files to `/path/to/output`.

- **Adjusting Writer Cores**

  ```
  padcirc -W 4
  ```
  
  Dedicates 4 MPI processes to write ASCII output files.

- **Using Hotstart Files**

  ```
  padcirc -R
  ```
  
  Reads subdomain binary hotstart files for initialization in a parallel run.


For more information see - [ADCIRC Webpage Documentation](https://adcirc.org/home/documentation/generic-adcirc-command-line-options/)
