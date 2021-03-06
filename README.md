## Table of contents

* [Introduction](#introduction)
* [Requirements](#requirements)
* [Building fastDNA](#building-fastdna)
* [Command line interface](#command-line-interface)
   * [Full documentation](#full-documentation)
* [Python](#python)
* [References](#references)
   * [Continuous Embedding of DNA reads and application to metagenomics](#continuous-embedding-of-dna-reads-and-application-to-metagenomics)
   * [Enriching Word Vectors with Subword Information](#enriching-word-vectors-with-subword-information)
   * [Bag of Tricks for Efficient Text Classification](#bag-of-tricks-for-efficient-text-classification)
   * [FastText.zip: Compressing text classification models](#fasttextzip-compressing-text-classification-models)
* [License](#license)

## Introduction

[fastDNA](#continuous-embedding-of-dna-reads-and-application-to-metagenomics) is a library for classification of short DNA sequences.
It is adapted from the [fastText](https://fasttext.cc/) library.


## Requirements

Generally, **fastText** builds on modern Mac OS and Linux distributions.
Since it uses some C++11 features, it requires a compiler with good C++11 support.
These include :

* (g++-4.7.2 or newer) or (clang-3.3 or newer)

Compilation is carried out using a Makefile, so you will need to have a working **make**.


## Building fastDNA


```
$ git clone https://github.com/rmenegaux/fastDNA.git
$ cd fastDNA
$ make
```

This will produce object files for all the classes as well as the main binary `fastdna`.

For a trial run:
```
$ cd test
$ sh test.sh
```
This should train and evaluate a small model on the toy dataset provided. 

### DNA short read classification

In order to train a dna classifier using the method described in [1](#continuous-embedding-of-dna-reads-and-application-to-metagenomics), use:

```
$ ./fastdna supervised -input train.fasta -labels labels.txt -output model
```

where `train.fasta` is a FASTA file containing the full reference genomes and `labels.txt` is a text file containing the genome labels (one label per line).
This will output two files: `model.bin` and `model.vec`.

Once the model was trained, you can evaluate it by computing the precision and recall at k (P@k and R@k) on a test set using:

```
$ ./fastdna test model.bin test.fasta test_labels.txt n
```

where `test.fasta` is a FASTA file containing the DNA fragments to be classified, and `test_labels.txt` contains the labels for each of the fragments.

The argument `n` is optional, and is equal to `1` by default.

In order to obtain the n most likely labels for a set of reads, use:

```
$ ./fastdna predict model.bin test.fasta n
```

or use `predict-prob` to also get the probability for each label

```
$ ./fastdna predict-prob model.bin test.fasta n
```

Doing so will print to the standard output the n most likely labels for each line.
The argument `n` is optional, and equal to `1` by default.

If you want to compute vector representations of DNA sequences, please use:

```
$ ./fastdna print-sentence-vectors model.bin < text.fasta
```

This assumes that the `text.fasta` file contains the DNA sequences that you want to get vectors for.
The program will output one vector representation per line in the file.

You can also quantize a supervised model to reduce its memory usage with the following command:

```
$ ./fastdna quantize -output model
```
This will create a `.ftz` file with a smaller memory footprint. All the standard functionality, like `test` or `predict` work the same way on the quantized models:
```
$ ./fastdna test model.ftz test.fasta test_labels.txt
```
The quantization procedure follows the steps described in [3](#fasttextzip-compressing-text-classification-models). 


### Full documentation

Invoke a command without arguments to list available arguments and their default values:
```
$ ./fastdna supervised
Empty input or output path.

The following arguments are mandatory:
  -input              training file path
  -output             output file path

The following arguments are optional:
  -verbose            verbosity level [2]

The following arguments for the dictionary are optional:
  -minn               min length of char ngram [0]
  -maxn               max length of char ngram [0]
  -label              labels prefix [__label__]

The following arguments for training are optional:
  -lr                 learning rate [0.1]
  -lrUpdateRate       change the rate of updates for the learning rate [100]
  -dim                size of word vectors [100]
  -noise              mutation rate (/100,000)[0]
  -length             length of fragments for training [200]
  -epoch              number of epochs [5]
  -loss               loss function {ns, hs, softmax} [softmax]
  -thread             number of threads [12]
  -pretrainedVectors  pretrained word vectors for supervised learning []
  -loadModel          pretrained model for supervised learning []
  -saveOutput         whether output params should be saved [false]
  -freezeEmbeddings   model does not update the embedding vectors [false]

The following arguments for quantization are optional:
  -cutoff             number of words and ngrams to retain [0]
  -retrain            whether embeddings are finetuned if a cutoff is applied [false]
  -qnorm              whether the norm is quantized separately [false]
  -qout               whether the classifier is quantized [false]
  -dsub               size of each sub-vector [2]
```

Defaults may vary by mode. (Word-representation modes `skipgram` and `cbow` use a default `-minCount` of 5.)

## Python

The python scripts require `numpy` and `scikit-learn` for evaluating predictions.

## References

### Continuous Embedding of DNA reads, and application to metagenomics

[1] R. Menegaux, J. Vert, [*Continuous Embedding of DNA reads, and application to metagenomics*](https://www.biorxiv.org/content/early/2018/05/31/335943)

```
@article{menegaux2018continuous,
  title={Continuous Embedding of DNA reads and application to metagenomics},
  author={Menegaux, Romain and Vert, Jean-Philippe},
  journal={bioRxiv preprint 335943},
  year={2018}
}
```
### Enriching Word Vectors with Subword Information

[2] P. Bojanowski, E. Grave, A. Joulin, T. Mikolov, [*Enriching Word Vectors with Subword Information*](https://arxiv.org/abs/1607.04606)

```
@article{bojanowski2016enriching,
  title={Enriching Word Vectors with Subword Information},
  author={Bojanowski, Piotr and Grave, Edouard and Joulin, Armand and Mikolov, Tomas},
  journal={arXiv preprint arXiv:1607.04606},
  year={2016}
}
```

### Bag of Tricks for Efficient Text Classification

[3] A. Joulin, E. Grave, P. Bojanowski, T. Mikolov, [*Bag of Tricks for Efficient Text Classification*](https://arxiv.org/abs/1607.01759)

```
@article{joulin2016bag,
  title={Bag of Tricks for Efficient Text Classification},
  author={Joulin, Armand and Grave, Edouard and Bojanowski, Piotr and Mikolov, Tomas},
  journal={arXiv preprint arXiv:1607.01759},
  year={2016}
}
```

### FastText.zip: Compressing text classification models

[4] A. Joulin, E. Grave, P. Bojanowski, M. Douze, H. Jégou, T. Mikolov, [*FastText.zip: Compressing text classification models*](https://arxiv.org/abs/1612.03651)

```
@article{joulin2016fasttext,
  title={FastText.zip: Compressing text classification models},
  author={Joulin, Armand and Grave, Edouard and Bojanowski, Piotr and Douze, Matthijs and J{\'e}gou, H{\'e}rve and Mikolov, Tomas},
  journal={arXiv preprint arXiv:1612.03651},
  year={2016}
}
```


* Contact: [romain.menegaux@mines-paristech.fr](mailto:romain.menegaux@mines-paristech.fr)

## License

fastText is BSD-licensed. An additional patent grant is also provided
