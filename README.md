Code for our ICLR'19 submission on Poincare GloVe. This repo is a fork of the [gensim repository](https://github.com/RaRe-Technologies/gensim).

Installation
------------

To set up the environment for the Poincare GloVe code follow the following steps:
- Install all the dependencies (e.g. `pip3 install Cython nltk annoy`). For the complete list of dependencies, check out the official gensim documentation.
- `git clone https://github.com/alex-tifrea/poincare_glove.git`
- `cd poincare_glove`
- `python3 setup.py develop`

This version has been tested under Python 3.6.

Documentation
-------------

For training and evaluating a model, we use run\_{word2vec, glove}.sh scripts.
Their usage is similar, so the following will focus only on GloVe.

**Training**:
To train a Vanilla (Euclidean) GloVe model, run the following:

`./run_glove.sh --train --root path/to/root --coocc_file path/to/coocc/file --vocab_file path/to/vocab/file --epochs 50 --workers 20 --restrict_vocab 200000 --chunksize 1000 --lr 0.05 --bias --size 100`

The root should be the folder that contains the repository folder as well (the
folder in which the `git clone` command was run). The coocc_file is a binary
file that contains co-occurrence triples in the format generated by the GloVe
preprocessing scripts
(https://github.com/stanfordnlp/GloVe/blob/master/src/cooccur.c). The
vocab\_file is a text file that contains the vocabulary and should have a format
similar to the one generated by
https://github.com/stanfordnlp/GloVe/blob/master/src/vocab_count.c.

For Poincare embeddings use something similar to the following:

`./run_glove.sh --train --root path/to/root --coocc_file path/to/coocc/file --vocab_file path/to/vocab/file --epochs 50 --workers 20 --restrict_vocab 200000 --lr 0.01 --poincare 1 --bias --size 100 --dist_func cosh-dist-sq`

The parameter `dist_func` specifies what function `h` to choose (where `h` is
the notation used in the paper). To see what each of the possible `dist_func`
options is doing, consult the code in glove_inner.pyx (function
`mix_poincare_similarity` and `poincare_similarity`)

For a Cartesian product of Poincare balls, the command changes a little bit:

`./run_glove.sh --train --root path/to/root --coocc_file path/to/coocc/file --vocab_file path/to/vocab/file --epochs 50 --workers 20 --restrict_vocab 200000 --lr 0.05 --poincare 1 --bias --size 100 --mix --num_embs 50 --dist_func cosh-dist-sq`

Here, `num_embs` specifies in how many small dimensional embeddings the large
vector of length `size` will be split.

A number of training options are available:
- `--init_pretrained` will initialize the embeddings from a pretrained model.
The pretrained model that will be used for a certain embedding size and setting
needs to be specified in the `glove_main.py` file, by changing the
`INITIALIZATION_MODEL_FILENAME` variable.
- `--no_redirect` will not redirect the output to a log file, but instead will
print it to `stdout`.
- `--no_eval` will not run the evaluation of the model at the end, after
training is complete.
- `--ckpt_emb` will save snapshots of the values of the embeddings for a number
of words, during training, so that they can be used to create 2D animations of
how the words' positions change. It is usually used for 2D (or 3D) embeddings,
that can be visualized.

**Evaluating**:
Evaluation for all model types works in a similar way. The command format is the
following:

`./run_glove.sh --eval --restrict_vocab 200000 --root path/to/root --model_file path/to/saved/model`

`path/to/saved/model` should point to the location of the model that is being
evaluated.

Some of the options that are allowed for evaluation are the following:
- `--agg` will not evaluate the model only on the target vectors, but instead it
will combine information from both the target (`w`) and the context (`c`)
vectors. For the Euclidean case this means simply adding the two, `w+c`. In the
hyperbolic case, we are instead taking the gyromidpoint between `w` and `c`.
- `--cosine_eval` is particularly useful when evaluating Poincare embeddings.
The default is to use the Poincare distance to assess similarity between words
and to select the closest word to the gyroparallel transported point according
to a query for analogy. This argument allows for using the cosine distance
instead.
- {`--cosadd`, `--cosmul`, `--hyp_pt`, `--distadd` etc} allow for using a
different technique for solving analogy queries, other than the default for each
model setup.

**Logs**:
During training/evaluation, some important information is saved in logs that can
later be used to generate plots or to debug and investigate the characteristics
of the trained embedding models.

In the folder `ROOT/logs` the model will save the progress that is recorded
during training, including information about the epoch loss, the vector norms
and the scores on some analogy and similarity benchmarks.

In `ROOT/eval_logs` you can find a persistant copy of the output generated by
the evaluation of a model.

## Pre-trained embeddings
Some of the embedding models presented in the paper are made available [here](https://polybox.ethz.ch/index.php/s/TzX6cXGqCX5KvAn).
The embeddings have been trained on a dump of the English Wikipedia containing 
1.4 billion tokens. For more details about the training setup, please consult 
the experiments section of our paper.

## References
If you find this code useful for your research, please cite the following paper:
```
@article{poincareglove,
  Author = {Alexandru Tifrea and Gary Bécigneul and Octavian-Eugen Ganea},
  Title = {Poincaré GloVe: Hyperbolic Word Embeddings},
  Year = {2018},
  Eprint = {arXiv:1810.06546},
}
```
