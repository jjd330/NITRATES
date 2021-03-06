# BatML

Some functions and scripts for doing a Maximum Likelihood analysis on Swift BAT data

# Current Analysis Scripts

`run_stuff_grb2.sh`
Used to run the full targeted analysis. 
Runs `mkdb.py`, `do_data_setup.py`, `do_full_rates.py`, then `do_manage2.py`
first arg is the trigger time, second arg is the Name of the trigger, and the optional third arg is the minimum duration to use

`mkdb.py`
Creates an sqlite DB that contains the trigger time and important file names
DB not used much in the analysis, used to be used to store results and is kind of a relic now

`do_data_setup.py`
Gathers the event, attitude, and enabled detectors files
Chooses which dets to mask, based on any hot or cold dets or any det glitches
Makes a "filtered" event file that has the events removed outside the usable energy range or far away from the analysis time
Also adds a GTI table to the event file for when it's not slewing and there's no multi-det glitches
Also makes a partial coding image if there's a usable set of HEASOFT tools

`do_full_rates.py`
Runs the full rates analysis to pick time bins as seeds for the analysis

`do_manage2.py`
Manages the rest of the analysis
Submits jobs to the cluster, organizes results, and emails out top results
First submits a job for the bkg fit to off-time data
Then submits several jobs for the split detector rates analysis
Gathers the split rates results and makes the final set of position and time seeds
Assigns which jobs will processes which seeds and writes them to rate_seeds.csv (for inside FoV jobs) and out_job_table.csv (for out of FoV jobs)
Submits several jobs to the cluster for both inside FoV and outside FoV analysis
Gathers results and emails out top results when all of the jobs are done

`do_bkg_estimation_wPSs_mp2.py`
Script to perform the bkg fit to off-time data
Ran as a single job, usually with 4 procs

`do_rates_mle_InOutFoV2.py`
Script to perform the split rates analysis
Ran as several single proc jobs

`do_llh_inFoV4realtime2.py`
Script to perform the likelihood analysis for seeds that are inside the FoV
Ran as several single proc jobs

`do_llh_outFoV4realtime2.py`
Script to perform the likelihood analysis for seeds that are outside the FoV
Ran as several single proc jobs


# Important Modules 

`LLH.py`
* Has class and functions to compute the LLH
* The `LLH_webins` class handles the data and LLH calculation for a given model and paramaters
  * It takes a model object, the event data, detmask, and start and stop time for inputs 
  * Converts the event data within the start and stop time into a 2D histogram in det and energy bins
  * Can then compute the LLH for a given set of paramaters for the model
  * Can do a straight Poisson likelihood or Poisson convovled with a Gaussian error

`minimizers.py`
* Funtctions and classes to handle numerically minimizing the NLLH
* Most minimizer objects are subclasses of `NLLH_Minimizer`
  * Contains functions for doing parameter transformations and setting bounds
  * Also handles mapping the tuple of paramter values used for a standard scipy minimizer to the dict of paramater names and values used by the LLH and model objects

`models.py`
* Has the models that convert input paramaters into the count rate expectations for each det and energy bin in the LLH
* The models are sub classes of the `Model` class
* Currently used diffuse model is `Bkg_Model_wFlatA`
* Currently used point source model is `Source_Model_InOutFoV`, which supports both in and out of FoV positions
* Currently used simple point source model for known sources is `Point_Source_Model_Binned_Rates`
* `CompoundModel` takes a list of models to make a single model object that can give the total count expectations from all models used

`flux_models.py`
* Has functions and classes to handle computing fluxes for different flux models
* The different flux model object as subclasses of `Flux_Model`
  * `Flux_Model` contains methods to calculate the photon fluxes in a set of photon energy bins
  * Used by the response and point source model objects
* The different available flux models are:
  * `Plaw_Flux` for a simple power-law
  * `Cutoff_Plaw_Flux` for a power-law with an exponential cut-off energy
  * `Band_Flux` for a Band spectrum 

`response.py`
* Contains the functions and objects for the point source model
* Most current response object is `ResponseInFoV2` and is used in the `Source_Model_InOutFoV` model

`ray_trace_funcs.py`
* Contains the functions and objects to read and perform bilinear interpolation of the foward ray trace images that give the shadowed fraction of detectors at different in FoV sky positions
* `RayTraces` class manages the reading and interpolation and is used by the point source response function and simple point source model

