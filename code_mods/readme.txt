ARCTIC SURFACE ROUGHNESS EXPERIMENT WORKFLOW

1) GETTING CESM CODE
Want to use the same version of the code as the CESM2-Large Ensemble so that we can use the CESM2-LE as a control. 
(CESM version cesm2.1.4-exp03 is the CESM2-LE code base, as per Nan Rosenblom email)
https://www.cesm.ucar.edu/models/cesm2/release_download.html
https://github.com/ESCOMP/CESM/releases/tag/cesm2.1.4-exp03
https://www.cesm.ucar.edu/models/cesm2/config/2.1.0/compsets.html

Thoughts on where to start runs.
Need to run 5 ensembles. Start first in 1940 or 1950:
/glade/p/cesmdata/inputdata/cesm2_init/b.e21.BHIST.f09_g17.CMIP6-historical.00?/
No restart files available in 1920. But ensembles 001, 002, 003, 004, 005, 006, 007, 008, 009 all have files available in 1950. Gives us spin up to 2015 when we can pertlim the other 4 ensembles and run all to 2100.

2) Putting all code and such at: /glade/work/duvivier/projects/arctic_cyclones/
>git clone -b cesm2.1.4-exp03 https://github.com/ESCOMP/CESM.git cesm2.1.4-exp03
>cd cesm2.1.4-exp03/
>./manage_externals/checkout_externals

3) HISTORICAL EXPERIMENT (1 ENS) SET UP
Create a case:
>cd /cime/scripts/
>./create_newcase --case /glade/work/duvivier/projects/arctic_cyclones/experiments/b.e21.BHIST.f09_g17.CMIP6-historical.rufmod --res f09_g17 --compset BHIST --project P93300065
>cd ../../../experiments/b.e21.BHIST…
Copy ice_atmo.F90 and ice_step_mod.F90 from /mods/ directory in the /experiments/ directory. This goes to SourceMods/src.cice/
Copy cam_diagnostics.F90 from /mods/ directory in the /experiments/ directory. This goes to SourceMods/src.cam/
Also copy user_nl_cice and user_nl_clm to the main case directory. Turned on f_CMIP to get sidragtop, but turning on f_cdn_atm didn’t really do anything so removed that. I also need to make a few changes to clm to get things running.
>./case.setup
>qcmd -- ./case.build 
>cp /glade/p/cesmdata/inputdata/cesm2_init/b.e21.BHIST.f09_g17.CMIP6-historical.011/1940-01-01/* /glade/scratch/duvivier/b.e21.BHIST.f09_g17.CMIP6-historical.rufmod/run/
For the first experiment we need to copy restart files from one of the CMIP runs starting in 1940. That will be 75 years total simulation to get through 2014. Then we’ll branch the other runs off that run using pertain at 2015 for the ssps once it’s spun up to the different roughness.
>./xmlchange STOP_OPTION=nyears,STOP_N=3,RUN_TYPE=hybrid,RUN_REFCASE=b.e21.BHIST.f09_g17.CMIP6-historical.011,RUN_REFDATE=1940-01-01,RUN_STARTDATE=1940-01-01,CLM_NAMELIST_OPTS=use_init_interp=.true.
Check env_run.xml and change any necessary vars for a branch run
During test found it took ~3 hrs to run 1 year and we have a total of 12 wall clock hours, so go in 3 hour chunks because 4 year might time out. blah.
>./case.submit

4) After first successful 3 year job, need to change some env_run.xml vars to get through 2014:
>./xmlchange CONTINUE_RUN=TRUE, RESUBMIT=25 (23)

5) After 2014:
Need to set up new cases that will be with SSPs. compset = BSSP370cmip6. Then from these for ensemble 1, just branch off the first run (above) but other 4 need to add to user_nl_cam: pertlim = 1.0e-14, 1.1e-14,1.2e-14,1.3e-14. Need different pertlim for each ensemble. (Double check with Cecile if this doesn’t work right away).
https://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/modelnl/nl_cam.html

6) FUTURE EXPERIMENT (5 ENS) SET UP
Future experiments - 5 ensembles starting in 2015
Create a case:
>cd /cime/scripts/
>./create_newcase --case /glade/work/duvivier/projects/arctic_cyclones/experiments/b.e21.BSSP370.f09_g17.rufmod.001 --res f09_g17 --compset BSSP370 --project P93300065
>cd ../../../experiments/b.e21.BSSP…
Copy ice_atmo.F90 and ice_step_mod.F90 from /mods/ directory in the /experiments/ directory. This goes to SourceMods/src.cice/
Copy cam_diagnostics.F90 from /mods_ssps/ directory in the /experiments/ directory. This goes to SourceMods/src.cam/
Also copy user_nl_cice, user_nl_cam, and user_nl_clm to the main case directory. 
NEED TO ADD PERTLIM TO user_nl_cam (See /glade/work/dbailey/cesm_runs for examples here)
X Ens1: no pertlim
X Ens2: pertlim = 1.0e-14
X Ens3: pertlim = 2.0e-14
X Ens4: pertlim = 3.0e-14
X Ens5: pertlim = 4.0e-14
>./case.setup
>qcmd -- ./case.build 
