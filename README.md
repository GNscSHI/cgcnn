# Suface-emphasized Crystal Graph Convolutional Neural Networks
(SE-CGCNN) This is the repository for our work on surface property prediction of crystals. The ideas of Crystal Graph Convolutional Neural Network, surface embedding and GradNorm algorithm (for multi-task learning) are combined to predict materials properties with higher accuracy and efficiency.

The package provides three major functions:
- Train a SE-CGCNN model with a customized dataset for single/multiple targets.
- Predict material properties of new crystals with a pre-trained SE-CGCNN model.
- Transfer the pre-trained model to customized datasets.
## Table of Contents

- [Important paper referenced](#Important-papar-referenced)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
  - [Define a customized dataset](#define-a-customized-dataset)
  - [Train a SE-CGCNN model](#train-a-se-cgcnn-model)
  - [Predict material properties with a pre-trained CGCNN model](#predict-material-properties-with-a-pre-trained-cgcnn-model)
- [Data](#data)
- [Authors](#authors)
- [License](#license)

## Important paper referenced
There are two important papers referenced for this work:
1. Crystal Graph Convolutional Neural Networks for an Accurate and Interpretable Prediction of Material Properties
   (https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.120.145301).
3. GradNorm algorithm for balancing the gradient loss of each task\
   (https://arxiv.org/abs/1711.02257)

##  Prerequisites

The following packages are required:

- [PyTorch](http://pytorch.org)
- [scikit-learn](http://scikit-learn.org/stable/)
- [pymatgen](http://pymatgen.org)

## Usage

### Define a customized dataset 

To input crystal structures to CGCNN, you will need to define a customized dataset. Note that this is required for both training and predicting. 

Before defining a customized dataset, you will need:

- [CIF](https://en.wikipedia.org/wiki/Crystallographic_Information_File) files recording the structure of the crystals that you are interested in
- The target properties for each crystal (not needed for predicting, but you need to put some random numbers in `id_prop.csv`)

You can create a customized dataset by creating a directory `root_dir` with the following files: 

1. `id_prop.csv`: a [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) file with **two or more** columns. The first column recodes a unique `ID` for each crystal, and the remaining columns record the values of target properties. If you want to predict material properties with `predict.py`, you can put any number in the remaining column. (The second column is still needed to identify the properties to predict.)

2. `atom_init.json`: a [JSON](https://en.wikipedia.org/wiki/JSON) file that stores the initialization vector for each element. An example of `atom_init.json` is `data/sample-regression/atom_init.json`, which should be good for most applications.

3. `ID.cif`: a [CIF](https://en.wikipedia.org/wiki/Crystallographic_Information_File) file that recodes the crystal structure, where `ID` is the unique `ID` for the crystal.

The structure of the `root_dir` should be:

```
root_dir
├── id_prop.csv
├── atom_init.json
├── id0.cif
├── id1.cif
├── ...
```

There are two examples of customized datasets in the repository: `data/sample-regression` for regression and `data/sample-classification` for classification. 

**For advanced PyTorch users**

The above method of creating a customized dataset uses the `CIFData` class in `cgcnn.data`. If you want a more flexible way to input crystal structures, PyTorch has a great [Tutorial](http://pytorch.org/tutorials/beginner/data_loading_tutorial.html#sphx-glr-beginner-data-loading-tutorial-py) for writing your own dataset class.

### Train a SE-CGCNN model

Before training a new CGCNN model, you will need to:

- [Define a customized dataset](#define-a-customized-dataset) at `root_dir` to store the structure-property relations of interest.

Then, in directory `cgcnn`, you can train a CGCNN model for your customized dataset by:

```bash
python main.py root_dir
```

You can set the number of training, validation, and test data with labels `--train-size`, `--val-size`, and `--test-size`. Alternatively, you may use the flags `--train-ratio`, `--val-ratio`, `--test-ratio` instead. Note that the ratio flags cannot be used with the size flags simultaneously. For instance, `data/sample-regression` has 10 data points in total. You can train a model by:

```bash
python main.py --train-size 6 --val-size 2 --test-size 2 data/sample-regression
```
or alternatively
```bash
python main.py --train-ratio 0.6 --val-ratio 0.2 --test-ratio 0.2 data/sample-regression
```

You can also train a classification model with label `--task classification`. For instance, you can use `data/sample-classification` by:

```bash
python main.py --task classification --train-size 5 --val-size 2 --test-size 3 data/sample-classification
```

After training, you will get three files in `cgcnn` directory.

- `model_best.pth.tar`: stores the CGCNN model with the best validation accuracy.
- `checkpoint.pth.tar`: stores the CGCNN model at the last epoch.
- `test_results.csv`: stores the `ID`, target value, and predicted value for each crystal in test set.

### Predict material properties with a pre-trained CGCNN model

Before predicting the material properties, you will need to:

- [Define a customized dataset](#define-a-customized-dataset) at `root_dir` for all the crystal structures that you want to predict.
- Obtain a [pre-trained CGCNN model](pre-trained) named `pre-trained.pth.tar`.

Then, in directory `cgcnn`, you can predict the properties of the crystals in `root_dir`:

```bash
python predict.py pre-trained.pth.tar root_dir
```

For instace, you can predict the formation energies of the crystals in `data/sample-regression`:

```bash
python predict.py pre-trained/formation-energy-per-atom.pth.tar data/sample-regression
```

And you can also predict if the crystals in `data/sample-classification` are metal (1) or semiconductors (0):

```bash
python predict.py pre-trained/semi-metal-classification.pth.tar data/sample-classification
```

Note that for classification, the predicted values in `test_results.csv` is a probability between 0 and 1 that the crystal can be classified as 1 (metal in the above example).

After predicting, you will get one file in `cgcnn` directory:

- `test_results.csv`: stores the `ID`, target value, and predicted value for each crystal in test set. Here the target value is just any number that you set while defining the dataset in `id_prop.csv`, which is not important.

## Data

To reproduce our paper, you can download the corresponding datasets following the [instruction](data/material-data).

## Authors

This software was primarily written by [Tian Xie](http://txie.me) who was advised by [Prof. Jeffrey Grossman](https://dmse.mit.edu/faculty/profile/grossman). 

## License

CGCNN is released under the MIT License.



