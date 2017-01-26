README for DeepCons
===================

INTRODUCTION
============
Comparative genomics has been very effective in finding functional elements across the human genome. However, understanding the functional roles of these sequences still remain a challenge, especially in noncoding regions. We present a deep learning approach, DeepCons, to understand sequence conservation. DeepCons is a convolutional neural network that is trained to classify conserved and non-conserved sequences. We show that the learned convolution kernels of DeepCons can capture rich information with respect to sequence conservation: 1) they match motifs such as CTCF, JUND, RFX3 and MEF2A that are known to be widely distributed within conserved noncoding elements, 2) they have positional bias relative to transcription start sites, transcription end sites and miRNA, and 3) they have strand bias relative to transcription end sites. DeepCons could also be used to score sequence conservation at nucleotide level resolution. We rediscovered known motifs within a given sequence by highlighting each nucleotide regarding their scores.


PREREQUISITES
=============
* Python (2.7). [Python 2.7.11](https://www.python.org/downloads/release/python-2711/) is recommended.

* [Numpy](http://www.numpy.org/)(>=1.10.4). 

* [Scipy](http://www.scipy.org/)(>=0.17.0). 

* [Theano](https://github.com/Theano/Theano/releases/tag/rel-0.8.2)(0.8.2).


DATA
====
All the data used for training can be downloaded from [here](https://cbcl.ics.uci.edu/public_data/DeepCons/). It contains three levels of conserved sequences: (1). the coordinates of the conserved/non-conserved sequences in bed format; (2). the randomly shuffled raw sequences in text format based on (1); (3). the sequences in numpy format ready for training based on (2). Please use binary mode to download the numpy files(e.g. wget command), directly ftp download using browser may corrupt the binary numpy file. After downloading all the numpy files, please put them in the same folder with `train_cnn.py`.

1. \*.bed files 
--------------
**cons.bed** contains the 887,577 conserved sequences with length in range of [30, 1000] used in the paper (hg19). 

**noncons.bed** contains the 887,577 non-conserved sequences obtained by randomly shuffling **cons.bed**.

2. \*.seq files
--------------
**data_tr.seq** contains the 1,415,154 raw sequences with labels for training obtained based on **cons.bed** and **noncons.bed**. It is a tab-delimited text file. The 1st column is the raw sequence, the 2nd column is the label with 1 = conserved, and 0 = non-conserved.

**data_va.seq** contains the 180,000 raw sequences with labels for validation. 

**data_te.seq** contains the 180,000 raw sequences with labels for testing. 

3. \*.npy files
--------------
Numpy files that are ready for running `train_cnn.py` obtained based on \*.seq files. For X_\*_float32.npy, its shape is as (N, 2020, 4). N is the number of sequences (e.g. 180,000 for X_va_float32.npy). 2020 is the length of the sequences, where the first 1-1000 bp is the original sequence padded with letter ''N'', 1001-1020 is a gap also represented as letter ''N'', and 1021-2020 is the reverse complement of the original sequence padded with letter ''N''. 4 is the dimension of one hot encoding for A,C,G,T.




TRAINING
========
Training DeepCons is done by run `train_cnn.py`. A training example using 1000 convolutional kernels of 10 bp length, 500 convolutional kernels of 20 bp length, 1500 hidden units, 0.25 dropout rate for convolutional layer and 0.5 dropout rate for hidden layer is by:
```
$ ./train_cnn.py 1000+500x1500_0.25x0.5_10x20 1000 500 1500 0.25 0.5 10 20
```
In which, **1000+500x1500_0.25x0.5_10x20** is the base name for all the output files.

OUTPUT
======
Each training instance will output 2 files. For example, by running 
```
$ ./train_cnn.py 1000+500x1500_0.25x0.5_10x20 1000 500 1500 0.25 0.5 10 20 
```
It outputs:

**1000+500x1500_0.25x0.5_10x20.json**, the json file describing the model architecture used by keras.

**1000+500x1500_0.25x0.5_10x20.hdf5**, the hdf5 file storing the model parameters used by keras.


DOWNSTREAM ANALYSIS
===================
Once the training is completed, several downstream analysis can be performed based on the trained model.

Motif
-----
Motif in MEME format can be obtained by running `filter2motif.py`
```
$ ./filter2motif.py X_tr_float32.npy 1000+500x1500_0.25x0.5_10x20
```
This will output **1000+500x1500_0.25x0.5_10x20.meme** based on the trained model **1000+500x1500_0.25x0.5_10x20.json/hdf5** and sequences in  **X_tr_float32.npy**. The MEME file used in the paper is in `results/meme/X_tr.meme`.

Salience
--------
Salience score for a given sequence can be obtained by running `salience.py`
```
$ ./salience.py X_tr_float32.npy 1000+500x1500_0.25x0.5_10x20 X_tr.salience
```
This will output **X_tr.salience** based on the trained model **1000+500x1500_0.25x0.5_10x20.json/hdf5** and sequences in  **X_tr_float32.npy**. It is a tab-delimited text file. The 1st column is the raw sequence, the 2nd column is the comma-delimited salience scores for each bp. Examples can be found in `results/salience/`. Visualization in html format can be further obtained by running `salience2html.py`, for example
```
$ ./salience2html.py CTCF.salience > CTCF.html
```
![alt tag](https://github.com/uci-cbcl/DeepCons/blob/master/results/salience/CTCF.png)

Other analysis
--------------
Other results presented in the paper, such as motif alignment to known motif databases using tomtom, positional bias analysis with respect to TSS, TES, miRNA using centrimo, motif clustering analysis using RSAT, can also be found in /results/
