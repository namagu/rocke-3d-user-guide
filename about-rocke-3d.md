From the [2024 ROCKE-3D Tutorial video](https://youtu.be/2hs17xiFEXc?list=PLpMmnV3HS7r3ryMbNOPynfSrZY4w7mHoB)

# Lineage

Base model is the GISS ModelE GCM (Schmidt+2014) that couples atmopshere (chemistry), ocean, land (vegetation), sea ice, land ice.

From that came ROCKE-3D GCM (Way+17) to model Earth's past to present, ancient Venus, ancient/modern Mars (with clouds), hypothetical Venus-like exoplanets around M dwarfs (Kepler-1649 b, Kane+18; Prox Cen B Del Genio+19), ancient lunar atmosphere, **obliquity hysteresis?!**, geothermal heating, spin-orbit resonances (Colose+2021), parameter sweeps (rotation rate, ocean types (slab vs. dynamic aka coupled), and insolations, Way+18).

# ROCKE-3D Limits
  * max timescale is 10k years (corresonds to many months wall time)
  * shallow atmospheres and mixing ratios for stratosphere/upper stratosphere only; no mini-Neptunes/Titan/gas giants, no atmospheric escape
  * 100 K < planet temperature < 400 K
  * static (not time-evolving) mean molecular weight of atmosphere
  * planet day > 12 hr


# Under the hood
ROCKE-3D model versions
  * planet_1.0
  * planet_2.0
  * planet_3.0 (TBD)
	
Antmosphere composition calculators 
  * NINT - fixed atmospheric compositions
  * OMA, MATRIX - gas-phase chemistry and aerosols; for trace gases,  can calculate atmospheric composition prognostically
	

Coordinates
  * uses a latitude-longitude grid system and vertical layers with column physics built in
  * separately treats atmosphere, ocean, land surface

#### Key GISS People:
  - Mike Way
  - Kostas
  - Tony
  - Igor




