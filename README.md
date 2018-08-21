# Opt_for_NNetwork
 code and data for optimization results, with model biological neuronal network

Here are the steps for generating the model data, using DA, and reading DA output.

1) Generate the model data that will be used in the DA procedure.  (Use "GenerativeModel...ipynb", and the injected current files.)

The three-neuron model is integrated forward via the Jupyter Notebook script "GenerativeModel...ipynb".  This script takes in 
three injected currents, one for each neuron: "injnew1_wStepsHigh2.txt", "injnew2_wStepsHigh2.txt", and "injnew3_wStepsHigh2.txt". 
It generates time series of all state variables of the model.  Six of those variables (out of the 27 total) are saved: time series of 
membrane voltage and calcium concentration for each of the three cells.  The data described in the paper are the three membrane
voltage traces (the Ca concentrations are also used, but those results are mentioned only briefly).

To generate the data, choose one of two sets of synaptic strengths: "quiescence" or "active" (WLC) - which are 
specified at the beginning of the script (and de-activate the set that you don't want to use).

Example data in this folder are as follows:

the three membrane voltage traces generated from the "quiescence" synapse strengths:
  V0_NaKLCaT_chaotic_Q_dt0.1_T800.0_18aug14_NOISY.txt
  V1_NaKLCaT_chaotic_Q_dt0.1_T800.0_18aug14_NOISY.txt
  V1_NaKLCaT_chaotic_Q_dt0.1_T800.0_18aug14_NOISY.txt
and three membrane voltage traces generated from the "active" (WLC) synapse strengths:
  V0_NaKLCaT_chaotic_WLC_dt0.1_T800.0_18aug14_NOISY.txt
  V1_NaKLCaT_chaotic_WLC_dt0.1_T800.0_18aug14_NOISY.txt
  V1_NaKLCaT_chaotic_WLC_dt0.1_T800.0_18aug14_NOISY.txt.
The filenames indicate that these come from an "NaKL-plus-CaT" neuron model, using a chaotic injected current, with a time step
of 0.1 ms, a total integration time of 800 ms, and that 1% noise was added to the resulting voltage trace prior to delivering the data
to the DA (Ipopt) procedure.  

Note that these example data are not the results of any pruning: the connectivity is all-to-all, and all neurons have leak, Na, K, and CaT ion channels.  THESE ARE THE DATA THAT WERE READ INTO THE DA PROCEDURE IN ORDER TO GENERATE THE MAIN RESULTS OF THE PAPER.

2) Download and install the Interior-point Optimizer (Ipopt).

Instructions are here:
https://projects.coin-or.org/Ipopt

3) Write an interface to work with Ipopt.

Instructions are on their website (above).  The interface can be written in any language; my previous group developed a 
C-and-Python-based interface, which is here:
https://github.com/yejingxin/minAone

4) Run Ipopt, and download the output. 

For each path, the output will be a .dat file of several hundred MB.  It is an mxn matrix.

The rows 'm' are the values of 'beta': i.e. the number of iterative re-weightings of the measurement-versus-model error of the cost function (see the description of "Annealing" in the main text of the paper).  An ideal value for beta is between 22 and 30; for all results presented in this paper, we found a value of 27 to yield the best estimates.

The number of columns is: (3 + (t x D) + p), where t is the number of timesteps in the data, D is the number of state variables, and p
is the number of parameters to be estimated.  The "3" at the start consists of the value of the cost function, and two flags
that are not important for our purposes.

5) Read the output .dat file from Ipopt ("Read-DA-Results...ipynb")

The information you want to extract is: the value of the cost function at each value of beta, and the values of the parameters at 
each value of beta.  The cost function climbs steeply with beta, and then levels off as you approach the deterministic limit (with model error much greater than measurement error).  You will find that choosing a beta value along the plateau of the cost function will 
yield the best estimates.

The script "Read-DA-Results...ipynb" plots the cost function versus beta, parameter values versus beta, and prints the precise
values of the parameters given a user-specified value of beta.
