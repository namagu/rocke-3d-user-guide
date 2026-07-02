A description of how the GCM codebase is structured from the [2024 ROCKE-3D Tutorial video](https://youtu.be/2hs17xiFEXc?list=PLpMmnV3HS7r3ryMbNOPynfSrZY4w7mHoB)


# Examples

There exists an [archive of publication supplements](https://portal.nccs.nasa.gov/GISS_modelE/ROCKE-3D/publication-supplements/) to base new simulations off of. 

Key componenets of each example:
  * Rundeck(s)  (end with `.R`)
  * boundary condition files (don't know yet what those look like)
  * model output (end with `.nc`, can be interpreted in Panoply)
	


# Important directories

.modelErc
`$HOME/.modelErc`

This lives next to your `.bashrc` file and tells R3D (ROCKE-3D) where to look for libraries, directories, compilers, etc.

rocke-3d home directory
`$HOME/modelE2_planet_1.0`

I'm running `planet_1.0` for now, but I could have a parallel directory for `planet_2.0` 

template rundecks
`$HOME/modelE2_planet_1.0/templates`

From everything I've seen, they really stress not to touch anything in this directory (and I really dislike that framing). I think its purpose is as the repository for the template rundecks that are referenced when you make a new rundeck of Earth or a new planet.

rundecks
`$HOME/modelE2_planet_1.0/decks

This is where the rundecks you make will live. 

source code
`$HOME/modelE2_planet_1.0/model` (and sub-directories)

