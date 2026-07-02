Hiiii this is my (very) un-edited guide to running ROCKE-3D on a high-performance computing (HPC) cluster. 

A few me-specific things that you will have to change if you're doing this for yourself:
  * I'm using HiperGator (hpg), which is the University of Florida HPC system, and ngl the [documentation](https://docs.rc.ufl.edu/) is not that great. Here's a [succint guide](https://jas000n.github.io/jekyll/2025-09-01-HiPerGator-Tutorial.html) from [Jas000n](https://github.com/Jas000n) that I've found useful. All the paths in my guide will be specific to the file structure for my research group's allocation in hpg.
  * I made a `conda` environment called `rocky` but I don't remember what's in it. Probably python.

Here are the official documentation bits from the NASA GISS folks for installing and running ROCKE-3D. 

- [NASA GISS ROCKE-3D homepage](https://simplex.giss.nasa.gov/gcm/ROCKE-3D/)  
- [ROCKE-3D 2024 Tutorial slides](https://simplex.giss.nasa.gov/gcm/ROCKE-3D/GISS_ROCKE-3D_tutorial_SLIDES_2024.pdf)  
- [ROCKE-3D 2025 Tutorial video](https://www.youtube.com/watch?v=2hs17xiFEXc&list=PLpMmnV3HS7r3ryMbNOPynfSrZY4w7mHoB&index=18)  
- [ROCKE-3D compiler installation (annotated by me)](https://docs.google.com/document/d/1xsPshwBE7r0rpkTiY51tNJ715gK7Aw39cyhzGMdowHs/edit?usp=sharing)  
- [ROCKE-3D planet\_1.0 local computer installation instructions (annoted by me)](https://docs.google.com/document/d/1Sn8tqfrG6A7L_BKjF8qmAs50EA8bnC6yAe5cISzekIc/edit?usp=sharing)  
- [ROCKE-3D cheat sheet (GISS)](https://docs.google.com/document/d/1iQDq0S5Ulqus6dbUUVVQQdP6k8P10vlMCuVqhn2sWpA/edit?usp=sharing)  
- [ROCKE-3D GISS publications page](https://portal.nccs.nasa.gov/GISS_modelE/ROCKE-3D/publication-supplements/)  
- [ROCKE-3D software paper](https://ui.adsabs.harvard.edu/abs/2017ApJS..231...12W/abstract)  
- [Crash solutions (GISS)](https://docs.google.com/document/d/1Hvy9vW9m8YiN8_wf7SnulWIEcoXjgHT88C_o73-sbqw/view?tab=t.0)


I really believe in sharing knowledge and alleviating code-induced suffering where possible, so if anything in here is opaque (or wrong, which is likely ngl), please _**let me know**_. If there's enough interest (like, more than two people basically who want it), I'm not opposed to starting a ROCKE-3D Discord or Tumblr or groupchat or whatever. 

My email is natalia.guerrero (at) ufl (dot) edu

# compiling a run

Start in `/blue/sarahballard/natalia.guerrero/modelE2_planet_1.0/decks`

Search for `openmpi` and which one is being used: `module avail openmpi` (imo this is is more useful than `module spider`)

- `ml conda`
- `conda activate rocky`
  
- `module load gcc`
- `module load openmpi/5.0.10`

  
`make clean; make -j setup RUN=P1SoM40_Test` 

This whil take a while as `make -j setup` takes a while. It will probably throw a lot of compiler Warnings, but that's ok.

# editing a run

Let's say we want to run Earth's rundeck for a *year*

rundeck: `E1oM20_Test.R`

### changing length of simulation 

```
 &INPUTZ
 YEARI=1900,MONTHI=12,DATEI=1,HOURI=0, ! pick IYEAR1=YEARI (default) or < YEARI
 YEARE=1900,MONTHE=12,DATEE=2,HOURE=0, KDIAG=13*0,
 ISTART=2,IRANDI=0, YEARE=1900,MONTHE=12,DATEE=1,HOURE=1,IWRITE=1,JWRITE=1,
/
```

Changing it so that the last line is:

```
 &INPUTZ
 YEARI=1900,MONTHI=12,DATEI=1,HOURI=0, ! pick IYEAR1=YEARI (default) or < YEARI
 YEARE=1900,MONTHE=12,DATEE=2,HOURE=0, KDIAG=13*0,
 ISTART=2,IRANDI=0, YEARE=1900,MONTHE=12,DATEE=1,HOURE=1,IWRITE=1,JWRITE=1,
/
```


## rundeck definitions
According to the [video](https://youtu.be/2hs17xiFEXc?list=PLpMmnV3HS7r3ryMbNOPynfSrZY4w7mHoB&t=2756), 

`DTsrc` **cannot** be changed during the run
`DT`: dynamic timestep;  can be changed

`YEARI, MONTHI, DATEI, HOURI` AND `YEARE,MONTHE,DATEE,HOURE` are the ymdh of the start and end of the run

`master_year`, set above in the file, sets which year it's drawing the climate data from, so it's running that climatology every year

`ISTART` is the start state (and can be an int 0(?)-8):
  * `ISTART=2`: cold (restart)
`YEARE, MONTHE,DATEE, HOURE` are the "setup hour" reads the initial conditions provided and then runs the first two timesteps and saves a restart file from which it continues running, so you run the model and then it stops and then you say continue running
  * `ISTART=8` is another one I've seen 

`Ndisk=480` is the write cadence to disk -- can boost it to 1000 or 2000 for a fast model, but want it to have smaller time steps for a slower model in case it crashes.

`KCOPY=1` saves the output to file
`KRSF=120` what interval of how many months for which to save a resetart file (RSF)  -- the more "formal" restart file; really big -- contains the full model state in 3D

`NIsurf` how many times you run your surface processes per physics timestep 


`I`  file is a transcript of the run itself; doesn't contain comments, just the input files, the parameters, and the start/end dates; this is the file that's being read when you start/stop the run, **NOT** the rundeck; simplest to modify when wanting to extend the run, etc. without having to recompile (but why would you do that? wouldn't you want the rundeck to be as accurate as possible??)


# running it

These are the scripts for just running it. We need to be a little fancier on hpg.....so we have to embed the following commands into an `sbatch` script, `go_E1oM20_Test.sh` but 
If you don't want to embed it in a script, you can just run it interactively:
`srun --account=sarahballard --qos=sarahballard-b ../exec/runE E1oM20_Test -cold-restart -np 1`


#### Using `go_E1oM20_Test.sh`


`-np`: number of processors; can do up to 44 (the number of latitudes minus 2) or 88 but it'll crash if the number is too big

Each HPG node has 32 cores, and it's one task per core, I think. 

So in the slurm script, 
  - `--nodes` should be some number fewer than the number of nodes the group has; Kyle has it set to `1`
  - `--ntasks` and `-np` should be the same number; `23` for E1oM20 (earth-like run)
  - made an educated guess for `--cpus-per-task`
	
```
#!/bin/bash

#SBATCH --qos=sarahballard-b		# uses our burst nodes
#SBATCH --error=E1oM20_TestTest.err	# error file
#SBATCH --output=E1oM20_TestTest.out # output file

#SBATCH --mail-type=ALL             # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=natalia.guerrero@ufl.edu     # Where to send mail

#SBATCH --nodes=1                   # max number of nodes (servers) allocated                   
#SBATCH --ntasks=23                 # number of MPI tasks (processes)
#SBATCH --cpus-per-task=1           # number of CPUs per task 
#SBATCH --time=05:00:00             # Wall time limit (days-hrs:min:sec)

#export PATH=/blue/sarahballard/ssagear/conda-env/photoenv/bin:$PATH
#export PYTHONPATH=$PYTHONPATH:/blue/sarahballard/ssagear/conda-env/photoenv/lib/python
export PATH=/blue/sarahballard/natalia.guerrero/modelE2_planet_1.0/model/mk_diags:$PATH
export PATH=/blue/sarahballard/natalia.guerrero/ModelE_Support/netcdf/bin:$PATH
export LD_LIBRARY_PATH=/blue/sarahballard/natalia.guerrero/ModelE_Support/netcdf/lib:$LD_LIBRARY_PATH

echo "Date              = $(date)"
echo "Hostname          = $(hostname -s)"
echo "Working Directory = $(pwd)"
echo ""
echo "Number of Nodes Allocated      = $SLURM_JOB_NUM_NODES"
echo "Number of Tasks Allocated      = $SLURM_NTASKS"
echo "Number of Cores/Task Allocated = $SLURM_CPUS_PER_TASK"

module load gcc openmpi

../exec/runE E1oM20_Test -np 23
``` 

Run with `sbatch go_E1oM20_Test.sh` (can't just do `./go_E1oM20_Test.sh` because it'll try to run it in the node we are using for our VSCode tunnel, which does not have enough processes for us) and check email for START, END, and any other messages. 

Check that it's running using `squeue -A sarahballard`


From Kyle Batra, adapted to our purposes:
![Screen Shot 2025-09-16 at 10.25.03 AM.png](:/a0b7bedd7c884a21a05f6f0303bb4374)


These are the commands from the documentation, but we are figuring out how they need to be modified for our purposes.
  (1) `../exec/runE P1SoM40_Test -cold-restart -np 2 ` writes the `partial` file which includes the run ID and that's the first timestep of the model that's run; useful for referencing to make sure everything looks ok, especially after changing the file -- can look at it in Panoply to confirm the initial conditions are what you expect
 (2) run `../exec/runE P1SoM40_Test -np 2` _without_ the `-cold-restart` command to continue the run 

# handling the outputs

Ok, go to the directory for the run, e.g. `/blue/sarahballard/natalia.guerrero/modelE2_planet_1.0/decks/E1oM20_Test

check the `*.PRT` file: `tail -f [runID]/*.prt
(it is big) 

The `monYear.zzzRUNID.nc` files, which will have all the simulated atmosphere and ocean data in them, are soft-linked here from where they live in `ModelE_Support/prod_decks/prod_runs/E1oM20_Test/` or similar.
  * `zzz` : type of file, e.g. `acc` (monthly accumulated diagnostics)
rsf:  big files for restart

#### Utilities: 
- `scaleacc` scales the accumulated files 
- `sumfiles` makes time-averaged acc files 
- `diffreport` compares netCDF files, e.g. comparing the output after using two different compilers
- `ncview` basic Linux functionality to quickly see gridded model output 
- `Panoply` viewing tool (GUI) 


To extract these files, run the netCDF tool `scaleacc` with option `all`:
```module purge
module load gcc openmpi netcdf
scaleacc PARTIAL.accE1oM20_Test.nc all 
```

You might have to add stuff to the `$LD_LIBRARY_PATH`: `export LD_LIBRARY_PATH=/blue/sarahballard/natalia.guerrero/ModelE_Support/netcdf/lib/:$LD_LIBRARY_PATH`


Running `scaleacc [.nc file] [option, e.g. all]` will make a bunch of new files:
```
PARTIAL.consrvE1oM20_Test.nc
PARTIAL.adiurnE1oM20_Test.nc  PARTIAL.hdiurnE1oM20_Test.nc
PARTIAL.agcE1oM20_Test.nc     PARTIAL.icijE1oM20_Test.nc
PARTIAL.aijE1oM20_Test.nc     PARTIAL.ijhcE1oM20_Test.nc
PARTIAL.aijkE1oM20_Test.nc    PARTIAL.oijE1oM20_Test.nc
PARTIAL.aijlE1oM20_Test.nc    PARTIAL.oijlE1oM20_Test.nc
PARTIAL.aijmmE1oM20_Test.nc   PARTIAL.oijmmE1oM20_Test.nc
PARTIAL.ajE1oM20_Test.nc      PARTIAL.ojlE1oM20_Test.nc
PARTIAL.ajlE1oM20_Test.nc     PARTIAL.olnstE1oM20_Test.nc
PARTIAL.aregE1oM20_Test.nc    PARTIAL.otjE1oM20_Test.nc
```

General key to `zzz` segment of filename:

`a` atmosphere
`o` ocean

`i` longitude
`j` latitude
`l` altitude

`k` pressure
`t` tracers
`i` ice 

(longer list of file types on slide 44-45 of [2024 Tutorial Slides](https://simplex.giss.nasa.gov/gcm/ROCKE-3D/GISS_ROCKE-3D_tutorial_SLIDES_2024.pdf))

Go to local machine and rsync the files you want:

`rsync -av --include="filepatterntoinclude" --exclude="*" username@rsync.rc.ufl.edu:/path/to/directory /path/to/local/directory` 

```
cd /local/directory/here/E1oM20_Test_nc
rsync -av --include="PARTIAL*nc" --exclude="*" natalia.guerrero@rsync.rc.ufl.edu:/blue/sarahballard/natalia.guerrero/modelE2_planet_1.0/decks/E1oM20_Test/ .
```

