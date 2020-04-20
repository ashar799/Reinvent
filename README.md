The application in this repository is described in the paper: "REINVENT 2.0 – an AI tool for de novo drug design"
=================================================================================================================

Usage
-----

1. Templates for inputs are provided in `reinvent/data/examples/templates` folder. Concrete examples for a quick start are provided in **[ReinventCommunity](https://github.com/MolecularAI/ReinventCommunity)**.
2. There are templates for 6 running modes. Each running mode can be executed by "python input.py some_running_mode.json" after activating the environment.
Templates have to be edited before using. The only thing that needs modification for a standard run are the file and folder paths. Most running modes produce logs that can be monitored by `tensorboard`, see below.
   * Logging folder is defined by setting a valid path to the "logging_path" field in json. This is required for all running modes.
3. Running modes:
   * Sampling: `sampling.json` can be used to start sampling. Requires a generative model as an input and produces a file that contains SMILES. We provide a generative model "reinvent/data/augmented.prior". Alternatively focused Agents generated by transfer learning or reinforcement learning can be sampled as well.
   * Transfer Learning (TL): `transfer_learning.json` is the relevant template and it can be used to focus the general prior towards a narrow chemical space by training on a representative sample of SMILES provided by user. Requires as an input a list of SMILES (example format in "reinvent/data/smiles.smi") and the generative model "reinvent/data/augmented.prior". The result will be a set of generative Agent checkpoints produced after each epoch of training and a final focused Agent. Inspect the `tensorboard` logs to estimate which Agent has the level of focusing that you prefer.
   * Reinforcement Learning (RL): Use `reinforcement_learning.json` as a template. The general input requires paths for both Agent and Prior generative models (in "reinforcement_learning" section of the JSON files). Both can be the same model provided by us "reinvent/data/augmented.prior" or alternatively the user can provide a focused Agent generated by TL. The output is a focused generative model and "scaffold_memory.csv" file which contains the best scoring SMILES during the RL run. The output folder is defined by setting a value for "resultdir". The scoring function object "scoring_function" can be either "name": "custom_product" or "name": "custom_sum". Scoring function has a list of parameters "parameters":[] which may contain any number of component objects. The current template example offers 5 components: a QED score, Matching Substructure (MS), Custom Alerts (CA) and 2 Predictive Property (PP) components. The PP components require setting either a classification ("reinvent/data/drd2.pkl") or regression ("reinvent/data/Aurora_model.pkl") model paths.

Tutorials / `jupyter` notebooks
-----
There is another repository containing useful `jupyter` notebooks related to `REINVENT` called **[ReinventCommunity](https://github.com/MolecularAI/ReinventCommunity)**. Currently, it contains notebooks to illustrate the Reinforcement Learning mode and the score transformation functionality (including how to find appropriate parameters for a component). Note, that the latter uses a different `conda` environment to execute.

Available components
-----
The scoring function is built-up from components, which together define the "compass" the Agents use to navigate the chemical space and suggest chemical compounds. Currently, there are the following components available:
* `PREDICTIVE PROPERTY`: Descriptor-based models to predict e.g. activity against a given target or solubility. Uses `scikit-learn` models (interface). Works with both classification and regression models.
* `TANIMOTO SIMILARITY`: Requires a user defined set of SMILES and returns the highest similarity score to the provided set.
* `JACCARD DISTANCE`: Requires a user defined set of SMILES and returns the lowest distance score to the provided set.
* `MATCHING SUBSTRUCTRE`: This is a penalty component to bias towards generating certain (sub-)structures. Requires a user defined set of SMARTS. Returns 1 if there is a substructure match and 0.5 otherwise.
* `CUSTOM ALERTS`: This is a penalty component to avoid generating certain (sub-)structures. Requires a user defined set of SMARTS patterns indicating unwanted moieties. Returns 0 if there is a match and 1 otherwise.
* `QED SCORE`: Uses the QED implementation in `RDKit`[link](http://rdkit.org/docs/source/rdkit.Chem.QED.html).
* `MOLECULAR WEIGHT`: Physico-chemical property calculated by `RDKit`[link](https://www.rdkit.org/docs/source/rdkit.Chem.Descriptors.html).
* `TPSA`: Physico-chemical property calculated by `RDKit`[link](https://www.rdkit.org/docs/RDKit_Book.html#implementation-of-the-tpsa-descriptor).
* `ROTATABLE BONDS`: Physico-chemical property calculated by `RDKit`[link](https://www.rdkit.org/docs/Cookbook.html#contiguous-rotable-bonds).
* `NUMBER OF HYDROGEN BOND DONOROS`: Physico-chemical property calculated by `RDKit`[link](https://www.rdkit.org/docs/source/rdkit.Chem.Lipinski.html).
* `NUMBER OF RINGS`: Physico-chemical property calculated by `RDKit`[link](https://www.rdkit.org/docs/source/rdkit.Chem.rdMolDescriptors.html).
* `SELECTIVITY`: If the aim is to optimize activity against one target while reducing activity agains another, i.e. to increase compound's selectivity, this component can be used. Uses two [scikit-learn](https://scikit-learn.org/stable/) models. Works with both classification and regression models. One model is predicting the target activity and the other is providing an off-target prediction. The score is reflecting a user defined activity gap between the target and the off-target predictions.

-------------------------------------------------
To use `tensorboard` for logging:

   1. To launch `tensorboard`, you need a graphical environment. Write:
       `tensorboard --logdir "path to your log output directory" --port=8008`
       This will give you an address to copy to a browser and access to the graphical summaries from `tensorboard`.

   2. Further command-line parameters can be used to change the amount of scalars, histograms, images, distributions and graphs shown, e.g.:
        `--samples_per_plugin=scalar=700, images=20`

Installation
-------------

1. Install Anaconda / Miniconda
2. Clone the repository
3. Open terminal, go to the repository and generate the appropriate environment:
    conda env create -f reinvent_shared.yml
4. (Optional) To set environmental variables (currently not needed), for example a license:
   On the command line, first:

       cd $CONDA_PREFIX
       mkdir -p ./etc/conda/activate.d
       mkdir -p ./etc/conda/deactivate.d
       touch ./etc/conda/activate.d/env_vars.sh
       touch ./etc/conda/deactivate.d/env_vars.sh

   then edit ./etc/conda/activate.d/env_vars.sh as follows:

       #!/bin/sh
       export SOME_LICENSE='/path/to/your/license/file'

   and finally, edit ./etc/conda/deactivate.d/env_vars.sh :

       #!/bin/sh
       unset SOME_LICENSE
5. Activate environment: conda activate reinvent_shared.v2.1
6. (Optional) In the project directory, in ./configs/ create the file `config.json` by copying over `example.config.json` and editing as required.
   In the current version this is only relevant for the unit tests.
7. Use the tool.


Tests - currently the number of tests is significantly reduced for this repo.
-----
Before running unit tests make sure to set up your `config.json`. Use the provided `configs/example.config.json`, rename it to `configs/config.json` and update the `USER_NAME` and `MAIN_TEST_PATH` variables. This file will not be version-controlled.
The tests can be executed using `Unittest`:
```
python -m unittest
```

Or using Pytest:
```
python -m pytest unittest_reinvent
```

Integration tests are decorated with `@pytest.mark.integration`. You can easily skip integration tests using `pytest` mark expression (`-m` argument):
```
python -m pytest -m "not integration" --strict-markers unittest_reinvent/
```
