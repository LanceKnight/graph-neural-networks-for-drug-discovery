# Property Prediction with Neural Networks on Raw Molecular Graphs


## Install dependencies

The code requires torch, rdkit and sklearn. Create a conda environment and install them for example by doing:

    $ conda create python=3.6.8 -p ~/gnnenv
    $ source activate ~/gnnenv/
    $ conda install pytorch=1.0.1 cudatoolkit=9.0 -c pytorch
    $ conda install rdkit=2018.09.1.0 -c rdkit
    $ conda install scikit-learn=0.20.2

Note that the above versions are the very specific ones I used. It may well be possible to relax the constraints. If you already have a conda environment with the three packages, you could first try to run the code with that.

If you want to use the tensorboard logging option you also need to install tensorboardX in your torch environment and tensorboard in any environment.


## Training

To see available models, do:

    $ python train.py -h

To see all options available when training for example the GGNN model, do:

    $ python train.py GGNN -h

A command line for training a model with some specific options may look like:

    $ python train.py ENNS2V --train-set toydata/piece-of-tox21-train.csv.gz --valid-set toydata/piece-of-tox21-valid.csv.gz --test-set toydata/piece-of-tox21-test.csv.gz --loss MaskedMultiTaskCrossEntropy --score roc-auc --s2v-lstm-computations 9 --out-hidden-dim 150 --logging more --epochs 20 --learn-rate 0.0001

To run a training session on your own data files, study the examples in toydata/ to understand how to format them. For classification data -1 represents negative, 0 missing and +1 positive.


## Using a saved model for prediction on new compounds

Use the --savemodel flag when starting the training:

    $ python train.py GGNN --epochs 3 --train-set toydata/piece-of-tox21-train.csv.gz --valid-set toydata/piece-of-tox21-valid.csv.gz --test-set toydata/piece-of-tox21-test.csv.gz --savemodel

After training is finished, a file in savedmodels/ will contain the model in the state it was when it showed the highest validation scores. Use it to make predictions on new data by doing:

    $ python predict.py --modelpath savedmodels/GGNN2019-02-22\ 12\:16\:17.432742 --score roc-auc --datapath toydata/piece-of-tox21-train-for-prediction.csv.gz

(Replace GGNN2019-02-22\ 12\:16\:17.432742 with your newly saved file.) Note that the same --score argument as used when training need to be supplied for correct output scaling. The predictions are printed to stdout in csv format. To store it to a file, add e.g. > predictions.csv to the end of the command.


## What are good hyperparameters?

For some hints, see the comments to the default hyperparameter dictionaries in train.py.


## Command line examples

Regression:

    $ python train.py ENNS2V --train-set toydata/piece-of-esol.csv.gz --valid-set toydata/piece-of-esol.csv.gz --test-set toydata/piece-of-esol.csv.gz --loss MSE --score RMSE --s2v-lstm-computations 9 --out-hidden-dim 150 --logging more --epochs 20 --learn-rate 0.0001

Submitting a job to slurm (if available):

    $ PYTHONUNBUFFERED=1 srun -t 60 -c 2 --mem 20g -p gpu --gres gpu:1 python train.py AttentionGGNN --cuda

--mem 20g is conservative enough to never run out of memory. -t depends entirely on dataset, epochs and model.


## Tests

To run all tests, do:

    $ python -m unittest discover --verbose

For a specific one, do:

    $ python -m unittest tests.test_example --verbose
