# ADCIRC User Guide { #adcirc_user_guide }

The ADCIRC (ADvanced CIRCulation) model is a system of computer programs often used in Coastal Engineering Storm Surge research for solving time dependent, free surface circulation and transport problems in two and three dimensions.
These programs utilize the finite element method in space allowing the use of highly flexible, unstructured grids.
ADCIRC is often coupled with The wind wave model SWAN (Simulating WAves Nearshore), especially in storm-surge applications where wave radiation stress can have important effects on ocean circulation and vice cersa.
Typical research topics include:

<ul>
	<li>prediction of storm surge and flooding</li>
	<li>modeling tides and wind driven circulation</li>
	<li>larval transport studies</li>
	<li>near shore marine operations</li>
	<li>dredging feasibility and material disposal studies</li>
</ul>

The following user guide gives a brief overview of ADCIRC and how it and supporting programs can be run on DesignSafe.

## ADCIRC Applications { #adcirc_apps }

ADCIRC is a suite of Fortran programs, for either parallel or serial execution.
The main components comprise of:

<ul>
	<li>adcirc - Non parallelized version of ADCIRC. 
	<ul>
		<li>This version of the application is ideal for smaller simulations, and runs on a single node on Frontera. Runtimes are subject to current wait times in the Frontera job queue.</li>
		<li>There is options to run the serial `adcirc` program within DesignSafe via the ADCIRC Interactive VM.</li>
	</ul>
	</li>
	<li>padcirc - Parallelized version of ADCIRC.
	<ul>
		<li>This is the Parallel version of the ADCIRC application and uses multiple compute nodes on TACC's Frontera or Lonestar6 HPC resource and is ideal for larger simulations. Runtimes are subject to current wait times in the HPC job queues.</li>
		<li>Within DesignSafe, `padcirc` simulations can be run within the ADCIRC Interactie VM, in the HPC JupyterHub, and via the TACC HPC queues on TACC's Fontera, Stampede3, and Lonestar6 HPC resources.</li>
	</ul>
	</li>
	<li>adcswan/padcswan - Serial/Parallelized versions of ADCIRC coupled with SWAN
	<ul>
		<li>The tightly coupled SWAN + ADCIRC paradigm allows both wave and circulation interactions to be solved on the same unstructured mesh resulting in a more accurate and efficient solution technique. </li>
		<li>This version of the application uses multiple nodes on TACC's Frontera or Lonestar6 HPC resource and is ideal for larger simulations. Runtimes are subject to current wait times in the HPC job queues.</li>
	</ul>
	</li>
    <li>ADCPREP
    <ul>
        <li>adcprep is a utility program that prepares input files for PADCIRC & PADCSWAN simulations. It partitions the mesh accross each parallel process, distributing the necessary input files, such as fort.15, fort.14, and fort.13, through a user-friendly interface.</li>
        <li>Note this utility only needs to be run when running the parallel version of ADCIRC and ADCIRC+SWAN.</li>
    </ul>
    </li>
</ul>

Along with the above programs, commonly used utilities used in conjunction with ADCIRC include:

- FigureGen - A fortran program for visualizing ADCIRC inputs and outputs over the grid. Has a variety of capabilities, and can be run within DesignSafe as a stand-alone app or through the Interactive ADCIRC VM. See the [FigureGen Document](../../visualization/figuregen.md) for more information.
- Kalpana - A python package for visualizign ADCIRC inputs/outputs and converting them into shapefiles and google kmz files for visualization in [QGIS](../../visualization/qgis.md). Kalpana can also be run through the Interactive ADCIRC VM, or as a standalone application. See the [Kalpana Documentation](../../visualization/kalpana.md) for more information.


### Decision Matrix for ADCIRC Applications { #adcirc_decision }

Deciding which DesignSafe application to run depends on your problem domain and size.
In general, the serial adcirc application is only used for testing and benchmarking, as most problems of interest require large grids.
The easiest way to determine the size of your ADCIRC problem is to measure it in terms of the # of finite elements in your grid.
This can be found at the top of the fort.14 file (see input files for more information):

```bash
❯ head fort.14  -n 5
 Quarter Annular Grid - Example 1           ! ALPHANUMERIC DESCRIPTOR FOR GRID FILE                   
 96  63                                     ! NE,NP - NUMBER OF ELEMENTS AND NUMBER OF NODAL POINTS   
    1     60960.0         0.0    3.0480     ! NODE NO., X, Y, DEPTH
    2     76200.0         0.0    4.7625
    3     91440.0         0.0    6.8580
```
*The Quarter Annular Grid Example can be found in the CommunityData folder at ``CommunityData/Use Case Products/ADCIRC/adcirc/adcirc_quarterannular-2d'*

For example, for the common benchmark test case invovling a hypothetical quarter annular grid, we can see that the problem size is of size 96 finite elements.
This test is more than ok to run using the regular ADCIRC version, and no ``adcprep`` prior run is required.
For cases when using parallel processing, the main deciding factor can be how many parallel processes to use.
Scaling studies have shown that targeting about 2000 nodes per process is ideal for these scenarios.
Thus the following table can be helpful for deciding where and when to run each application.

| # Elements       | ADCPREP? | # Nodes per Process | Sequential ADCIRC | Parallel ADCIRC | SWAN + ADCIRC |
|------------------|----------|---------------------|-------------------|-----------------|---------------|
| < 1000           | No       | 1                   | ✅                | 🔄 [^1]         | 🔄 [^2]       |
| 1000 - 1 million | Yes      | 2000                | ❌                | ✅              | ✅            |
| > 1 million      | Yes [^3] | 2000                | ❌                | ✅ [^4]         | ✅            |

- ✅: Recommended for this scenario.
- ❌: Not recommended or not yet available.
- 🔄: Viable under certain conditions or for certain job sizes. Please refer to footnotes.

[^1]: For models with fewer than 1000 elements, parallel ADCIRC may not be necessary, but it can be used depending on the available computational resources.
[^2]: For small models, SWAN + ADCIRC might be more resource-intensive than required unless specific wave dynamics need to be resolved.
[^3]: For very large models, ADCPREP can take a significant amount of time due to the decomposition of large grids. It is recommended that this data be saved and reused when possible to avoid the need for repeated decomposition.
[^4]: For very large models with complex wave-current interactions, SWAN + ADCIRC in parallel is the recommended approach.

## ADCIRC ON DesignSafe

DesignSafe Offers a variety of platforms on which to run and test ADCIRC related applications.
Behing the scenes power it is all powered by the Tapis API which connects the compute resources with the analysis environments.
In the context of how ADCIRC can be run, the two important things to keep in mind is (1) Where is the computation running and (2) Through what interface am I interacting with this compute platform.
At a high level, the compute platforms, from most powerful to least, that DesignSafe offers for computation are:

1. High Performance Computing (HPC) Job Queues - Job queues that are configured to handle jobs requiring multiple compute nodes, with GPU compute nodes also available. ADCIRC can be run on HPC job queues in the following manner:

	- HPC ADCIRC Applications - By using the pre-configured HPC applications either via the Web-Portal or throug the Tapis API. These HPC applications run pre-built version of ADCIRC on inputs that you can upload to your MyData on DesignSafe.
	- HPC Jupyter Instances - These are jupyter images running on an HPC queue, and can provide GPU support. For the moment, no native ADCIRC applications are supported in the HPC Jupyter instances, but ADCIRC can be installed in these environments.
	- TACC- By requesting a specific allocation on TACC. Usually this is done if more resources are required for larger runs. Please open a ticket if your use case requires more than the resources provided by the pre-configured HPC applications. 
	For more information on requesting HPC allocations, please refer to the [HPC Allocations documentation](../../advanced/hpcallocations.md#intro).

2. JupyterHub Images - These run on dedicated VMs, so they can handle more computation, but not as much as the HPC job queues that have access to multiple nodes for massively parallel jobs.
3. Interactive VMs - These run on shared VM resources, and therefore handle the lightest form of computations. The Interactive VM is launched from the web-portal, and offers a convenient environment for testing ADCIRC applications before running in a production environment.

### ADCIRC Through the Interactive VM

The Interactive VM is a docker image running on a shared VM with ADCIRC and supporting utilities pre-built for easy testing and development of ADCIRC related applications within the DesignSafe environment.

#### Advantages and Disadvantages of Interactive ADCIRC VM

A few advantages of using the ADCIRC VM include:

1. No queue wait time - Don't have to wait in the HPC queue to test input files.
2. Pre-compiled versions of ADCIRC and supporting utilities such as FigureGen and Kalpana.
3. Convenient Jupyter Lab interface, with plugins for github repo managemen, code formatting, and more.

Disadvantages include:

1. VM runs on a shared resource - Can be slow if lots of users are using the VM.
2. Not as large compute power - To simulate hurricanes at high-fidelity, ADCIRC needs to run on very large grids, which may take too long to run in the VM. Furthermore memory requirements for plotting and visualizing the grids and associated data may be too large for the interactive VM.

**Overall, the interactive VM is meant to be as a testing and learning environment. It is ideal to configure and test smaller runs of large jobs before submitting to the HPC queue to verify inputs/outputs are configured correctly for ADCIRC and supporting programs.**

#### Getting Started

You can access the interactive VM  via the DesignSafe-CI workspace by selecting "Workspace" &gt; "Tools &amp; Applications" &gt; "Simulation" &gt; "ADCIRC" &gt; Select "Jupyter" &gt; "Interactive VM for ADCIRC" to start the interactive VM.

![Interactive VM Selection](./images/interactive-vm-select.png)
*Selecting the Interactive VM for ADCIRC*

The intereactive VM will spawn a JupyterLab instance for you on a shared VM not within the HPC queues, so wait time shoudl be minimal (all though may be a little longer if it's your first time).
After your job goes into the "Running" stage, a dialogue box should prompt you to connect to your instance.

![Interactive VM Connect](./images/interactive-vm-connect.png)
*Once Job is running and window appears, click on Connect*

Once you click connect you should see a familiar JupyterLab interface:

![VM Launcher](./images/launcher-screen.png)
*Jupyter Lab Interface Launcher screen provides Kalpana kernel and terminal environment with ADCIRC and FigureGen*

#### Example - Running an ADCIRC simulation

One way to run an ADCIRC simulation in the interactive VM is via the linux terminal available from the Jupyter Lab interface.
Open a new terminal from the launcher window, navigate to your MyData directory and create a new directory for your ADCIRC run.
We copy into this directory some example ADCIRC input files corresponding to the ADCIRC Shinnecock Inlet test case (see [ADCIRC data available on DesignSafe](./adcirc.md#adcirc-data-hosted-on-designsafe) for more example cases).

```bash
cd ~/work/MyData
cp -r ~/work/CommunityData/Use\ Case\ Products/ADCIRC/adcirc/adcirc_shinnecock_inlet .
```

**Serial Run**

Now from within this directory we can run the code in serial by simply running the ``adcirc`` command from the root directory containing the ADCIRC input files.

```bash
cd ~/work/MyData/adcirc_shinnecock_inlet
adcirc
```

You should see an output similar to:

![ADCIRC Output](./images/adcirc-example-output.png)
*Example ADCIRC output indicating the max elevation and maximum water velocity values and location at each time step.*

Note the outputs are created in the same directory as the inputs (see left folder bar in Jupyter Lab interface).

**Parallel Run**

To run the same simulation in parallel, we must first run adcprep to prep the files for a parallel run.
If we want to run the same simulation with four parallel processes, we must (from a clean simulation directory) run adcprep twice.

```bash
cd ~/work/MyData
cp -r ~/work/CommunityData/Use\ Case\ Products/ADCIRC/adcirc/adcirc_shinnecock_inlet .
adcprep
```

Note adcprep is an interactive program.

![adcprep first run](./images/adcprep-example.png)
*Example of running ``adcprep`` the first time to partition the mesh.*

On the first run through the you want to partition the mesh inputing in order:

1. Number of processes for parallel run - Be careful it does not exist available processes from an mpi-run.
2. Action to perform - Option 1 on the first run to partition the mesh, which must be done first.
3. Name of the fort.14 file - In our case the default name of fort.14

After partitioning the mesh, a ``partmesh.txt`` and ``metis_graph.txt`` file should be created.
On the second run, you will input in the following order:

1. Number of processes for parallel run - Be careful it does not exist available processes from an mpi-run.
2. Action to perform - Option 2 on the second run to prep the rest of the input files.

![adcprep second run - Prepall](./images/adcprep2-example.png)
*Example of running ``adcprep`` the second time to prep individual PE* run directories.*

Note how after the second adcprep run, PE* directories are created for the input/output files corresponding to each individual process.

After both runs of ``adcprep``, ``padcirc`` can now be run.
Note how it must be launched using the ``mpirun`` command, specifying the number of outputs.

```
mpirun -np 4 padcirc
```


### Running ADCIRC on HPC Resources

ADCIRC can be run on HPC resources at TACC through DesignSafe through the pre-configured HPC applications.
Currently all these are configured to run only on TACC's [Frontera supercomputer](https://tacc.utexas.edu/systems/frontera/).

| App ID								| App Name									 |
|---------------------------------------|--------------------------------------------|
| adcirc_netcdf_55_Frontera-55.01u4     | ADCIRC-V55 (Frontera)                      |
| padcirc_swan-net_frontera_v55-55.00u4 | PADCIRC SWAN (Frontera) - V55              |
| padcirc-frontera-55.01u4              | PADCIRC (Frontera) - V55                   |

Note while the web portal provides a convenient interface to submit HPC jobs, the Tapis API provides a more programmatic way to interact and launch jobs.
The corresponding Tapis application IDs of the web-portal apps avaialable are listed in the table above.
We will review how to run HPC jobs through both of these interfaces below.

#### Using the Web Portal

To access the DesignSafe

<ul>
	<li>Select the appropriate ADCIRC application from the Simulation tab in the Workspace.</li>
	<li>Locate your Input Directory (Folder) with your input files that are in the Data Depot and follow the onscreen directions to enter this directory in the form.</li>
	<li>For the Parallel versions, enter your Mesh File into the form (usually fort.14 file).</li>
	<li>Enter a maximum job runtime in the form. See guidance on form for selecting a runtime.</li>
	<li>Enter a job name.</li>
	<li>Enter an output archive location or use the default provided.</li>
	<li>For the Parallel versions, select the number of nodes to be used for your job. Larger data files run more efficiently on higher node counts.</li>
	<li>Click Run to submit your job.</li>
	<li>Check the job status by clicking on the arrow in the upper right of the job submission form.</li>
</ul>


#### Using Tapis

Coming soon...

#### Within HPC Jupyter

Coming soon...

## ADCIRC Reference

The below section provides a brief overview of more techinical aspects of ADCIRC for quick reference. 
Links to supporting documentation are included or can be found below in the [external documentation](./adcirc.md#external-documentation--external_docs).

### ADCIRC Command Line Options

{% include-markdown './adcirc_cli.md' %}

### ADCIRC Input Files

{% include-markdown './inputs/inputs.md' %}

### ADCIRC Outputs Files

{% include-markdown './outputs/outputs.md' %}

### ADCIRC Examples

{% include-markdown './examples/examples.md' %}

### ADCIRC Installation (Advanced)

{% include-markdown './installation.md' %}

## Resources and Documentation

The following sections provide further information on useful resources for using ADCIRC.

### ADCIRC Data Hosted on DesignSafe

A wealth of ADCIRC related data can be found already on DesginSafe, in both the CommunityData and published projects folder.
The following are a few notable locations to find already available data to use for ADCIRC simulations:

- Community Data
	- Use Case Products - ``CommunityData/Use Case Products/ADCIRC/adcirc``
	- App Examples - ``CommunityData/app_examples/``
- Notable ADCIRC Published Projects


### External Documentation { #external_docs }

There are a wide variety of ADCIRC resources on the web.
Below are a few to help navigate and learn more about ADCIRC.

- **[ADCIRC GitHub Page](https://github.com/adcirc)** - As of v54, the official central information hub for all things ADCIRC. Contains source code, utility programs, and issue tracking; a good place for developers and users interested in the staying up to date with the latest updates and developments in ADCIRC, along with bug fixes and issues. Useful links include:
	- [Issues](https://github.com/adcirc/adcirc/issues) - For reporting bugs, searching for common issues with ADCIRC, or asking questions/feature requests.
	- [Test Suite](https://github.com/adcirc/adcirc-testsuite) - Test suite of ADCIRC examples, used for testing new releases of ADCIRC.
- **[ADCIRC Official Website](https://adcirc.org)** - Older primary source for all things ADCIRC, including model description, capabilities, and latest updates. Useful Sub pages include:
	- [Input File Descriptions](https://adcirc.org/home/documentation/users-manual-v53/input-file-descriptions/)/[Output File Descriptions](https://adcirc.org/home/documentation/users-manual-v53/output-file-descriptions/) - Mostly correct for basic inputs/outputs, barring any changes since v54+.
	- [Parameter Definitions](https://adcirc.org/home/documentation/users-manual-v53/parameter-definitions)
	- [Example Problems](https://adcirc.org/home/documentation/example-problems/)
- **[ADCIRC Wiki](https://wiki.adcirc.org)** - Out of date, but some useful information on here.

### Other ADCIRC Utilities and Libraries

The ADCIRC community is vast, with utility libraries being developed at different institutions around the world.
Below we highlight a few other third-party ADCIRC utilities and libraries that are not currently supported on DesignSafe, but can be useful and may be supported in the future.

- [pyadcirc](https://github.com/UT-CHG/pyadcirc)
- [adcircpy](https://github.com/oceanmodeling/adcircpy)

Do you have an ADCIRC utility or library you'd like to add to the list? Open a ticket to contribute to the user guide!