# Opt_for_NNetwork
 code and data for optimization results, with model biological neuronal network

Here are the steps for generating the model data, using DA, and reading DA output.

* STEP 1: 
Generate the model data that will be used in the DA procedure.  (Use "GenerativeModel...ipynb", and the injected current files.)

The three-neuron model is integrated forward via the Jupyter Notebook script "GenerativeModel...ipynb".  This script takes in 
three injected currents, one for each neuron: "injnew1_wStepsHigh2.txt", "injnew2_wStepsHigh2.txt", and "injnew3_wStepsHigh2.txt". 
It generates time series of all state variables of the model.  Six of those variables (out of the 27 total) are saved: time series of 
membrane voltage and calcium concentration for each of the three cells.  The data described in the paper are the three membrane
voltage traces (the Ca concentrations are also used, but those results are mentioned only briefly).

To generate the data, choose one of two sets of synaptic strengths: "quiescence" or "active" (WLC) - which are 
specified at the beginning of the script (and de-activate the set that you don't want to use).

* Data in this folder are as follows:

the three membrane voltage traces generated from the "quiescence" synapse strengths: "V0_NaKLCaT_chaotic_Q_dt0.1_T800.0_18aug14_NOISY.txt", "V1_...", "V2...", 

and three membrane voltage traces generated from the "active" (WLC) synapse strengths: "V0_NaKLCaT_chaotic_WLC_dt0.1_T800.0_18aug14_NOISY.txt", V1...", "V2..."

The filenames indicate that these come from an "NaKL-plus-CaT" neuron model, given a chaotic injected current, with a time step
of 0.1 ms, a total integration time of 800 ms, and that 1% noise was added to the resulting voltage trace prior to delivering the data
to the DA procedure.  

Note that these example data are not the results of any pruning: the connectivity is all-to-all, and all neurons have leak, Na, K, and CaT ion channels.  THESE ARE THE DATA THAT WERE READ INTO THE DA PROCEDURE IN ORDER TO GENERATE THE MAIN RESULTS OF THE PAPER.

* STEP 2:
Download and install the Interior-point Optimizer (Ipopt).

Instructions are here:
https://projects.coin-or.org/Ipopt

* STEP 3:
Use an interface to work with Ipopt, or write your own.

Instructions are on their website (above).  The interface can be written in any language; my previous group developed a 
C-and-Python-based interface, which is here:
https://github.com/yejingxin/minAone

* STEP 4:
Run Ipopt, and download the output. 

For each path, the output will be a .dat file of several hundred MB.  It is an mxn matrix.

The rows 'm' are the values of 'beta': i.e. the number of iterative re-weightings of the measurement-versus-model error of the cost function (see the description of "Annealing" in the main text of the paper).  An ideal value for beta is between 22 and 30; for all results presented in this paper, we found a value of 27 to yield the best estimates.

The number of columns is: (3 + (t x D) + p), where t is the number of timesteps in the data, D is the number of state variables, and p
is the number of parameters to be estimated.  The "3" at the start consists of the value of the cost function, and two flags
that are not important for our purposes.

* STEP 5:
Read the output .dat file from Ipopt ("Read-DA-Results...ipynb")

The information you want to extract is: the value of the cost function at each value of beta, and the values of the parameters at 
each value of beta.  The cost function climbs steeply with beta, and then levels off as you approach the deterministic limit (with model error much greater than measurement error).  You will find that choosing a beta value along the plateau of the cost function will 
yield the best estimates.

The script "Read-DA-Results...ipynb" plots the cost function versus beta, parameter values versus beta, and prints the precise
values of the parameters given a user-specified value of beta.

* STEP 6: Run prediction, via the estimated parameters (and change the injected current to 'current_square.dat', if you want to look at 'quiescent' and 'active' mode).

Now use the generative model to make a prediction for future times.  To do this, replace the parameter values with the estimates 
from Ipopt.  And compare the resulting integration to the integration of the original ("true") model.  

If you want to look at the 'quiescence' versus 'active' modes of circuit activity described in the paper, you need to change the 
injected currents given to the neuron models.  That is: rather than using the chaotic-plus-step injected currents that were used 
to generate the data used in the DA procedure, use: "current_square.dat" for each neuron.  This you can specify in the first few 
lines of the generative script "GenerativeModel...ipynb".
