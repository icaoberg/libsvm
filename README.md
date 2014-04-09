* libsvm 

libsvm is a simple, easy-to-use, and efficient software for SVM
classification and regression. It solves C-SVM classification, nu-SVM
classification, one-class-SVM, epsilon-SVM regression, and nu-SVM
regression. It also provides an automatic model selection tool for
C-SVM classification. This document explains the use of libsvm.

Libsvm is available at

* http://www.csie.ntu.edu.tw/~cjlin/libsvm

Please read the COPYRIGHT file before using libsvm.

* Table of Contents

- Quick Start
- Installation and Data Format
- `svm-train` Usage
- `svm-predict` Usage
- `svm-scale` Usage
- Tips on Practical Use
- Examples
- Precomputed Kernels 
- Library Usage
- Java Version
- Building Windows Binaries
- Additional Tools: Sub-sampling, Parameter Selection, Format checking, etc.
- MATLAB/OCTAVE Interface
- Python Interface
- Additional Information

* Quick Start

If you are new to SVM and if the data is not large, please go to 
`tools' directory and use easy.py after installation. It does 
everything automatic -- from data scaling to parameter selection.

Usage: `easy.py training_file [testing_file]`

More information about parameter selection can be found in
`tools/README`.

* Installation and Data Format

On Unix systems, type `make` to build the `svm-train` and `svm-predict`
programs. Run them without arguments to show the usages of them.

On other systems, consult `Makefile` to build them (e.g., see
'Building Windows binaries' in this file) or use the pre-built
binaries (Windows binaries are in the directory `windows').

The format of training and testing data file is:

```
<label> <index1>:<value1> <index2>:<value2> ...
.
.
.
```

Each line contains an instance and is ended by a `'\n'` character.  For
classification, `<label>` is an integer indicating the class label
(multi-class is supported). For regression, `<label>` is the target
value which can be any real number. For one-class SVM, it's not used
so can be any number.  The pair `<index>:<value>` gives a feature
(attribute) value: `<index>` is an integer starting from 1 and `<value>`
is a real number. The only exception is the precomputed kernel, where
`<index>` starts from 0; see the section of precomputed kernels. Indices
must be in ASCENDING order. Labels in the testing file are only used
to calculate accuracy or errors. If they are unknown, just fill the
first column with any numbers.

A sample classification data included in this package is
`heart_scale`. To check if your data is in a correct form, use
`tools/checkdata.py` (details in `tools/README`).

Type `svm-train heart_scale`, and the program will read the training
data and output the model file `heart_scale.model`. If you have a test
set called heart_scale.t, then type `svm-predict heart_scale.t
heart_scale.model output` to see the prediction accuracy. The `output`
file contains the predicted class labels.

For classification, if training data are in only one class (i.e., all
labels are the same), then `svm-train` issues a warning message:
`Warning: training data in only one class. See README for details,'
which means the training data is very unbalanced. The label in the
training data is directly returned when testing.

There are some other useful programs in this package.

`svm-scale`:

This is a tool for scaling input data file.		

`svm-toy`:

This is a simple graphical interface which shows how SVM
separate data in a plane. You can click in the window to 
draw data points. Use "change" button to choose class 
1, 2 or 3 (i.e., up to three classes are supported), "load"
button to load data from a file, "save" button to save data to
a file, "run" button to obtain an SVM model, and "clear"
button to clear the window.

You can enter options in the bottom of the window, the syntax of
options is the same as `svm-train'.

Note that "load" and "save" consider dense data format both in
classification and the regression cases. For classification,
each data point has one label (the color) that must be 1, 2,
or 3 and two attributes (x-axis and y-axis values) in
[0,1). For regression, each data point has one target value
(y-axis) and one attribute (x-axis values) in [0, 1).

Type `make` in respective directories to build them.

You need Qt library to build the Qt version.
(available from http://www.trolltech.com)

You need GTK+ library to build the GTK version.
(available from http://www.gtk.org)

The pre-built Windows binaries are in the `windows'
directory. We use Visual C++ on a 32-bit machine, so the
maximal cache size is 2GB.

* Usage

** `svm-train` Usage

```
Usage: svm-train [options] training_set_file [model_file]
options:
-s svm_type : set type of SVM (default 0)
	0 -- C-SVC		(multi-class classification)
	1 -- nu-SVC		(multi-class classification)
	2 -- one-class SVM	
	3 -- epsilon-SVR	(regression)
	4 -- nu-SVR		(regression)
-t kernel_type : set type of kernel function (default 2)
	0 -- linear: u'*v
	1 -- polynomial: (gamma*u'*v + coef0)^degree
	2 -- radial basis function: exp(-gamma*|u-v|^2)
	3 -- sigmoid: tanh(gamma*u'*v + coef0)
	4 -- precomputed kernel (kernel values in training_set_file)
-d degree : set degree in kernel function (default 3)
-g gamma : set gamma in kernel function (default 1/num_features)
-r coef0 : set coef0 in kernel function (default 0)
-c cost : set the parameter C of C-SVC, epsilon-SVR, and nu-SVR (default 1)
-n nu : set the parameter nu of nu-SVC, one-class SVM, and nu-SVR (default 0.5)
-p epsilon : set the epsilon in loss function of epsilon-SVR (default 0.1)
-m cachesize : set cache memory size in MB (default 100)
-e epsilon : set tolerance of termination criterion (default 0.001)
-h shrinking : whether to use the shrinking heuristics, 0 or 1 (default 1)
-b probability_estimates : whether to train a SVC or SVR model for probability estimates, 0 or 1 (default 0)
-wi weight : set the parameter C of class i to weight*C, for C-SVC (default 1)
-v n: n-fold cross validation mode
-q : quiet mode (no outputs)
```

The k in the -g option means the number of attributes in the input data.

option -v randomly splits the data into n parts and calculates cross
validation accuracy/mean squared error on them.

See libsvm FAQ for the meaning of outputs.

** `svm-predict` Usage

```
Usage: svm-predict [options] test_file model_file output_file
options:
-b probability_estimates: whether to predict probability estimates, 0 or 1 (default 0); for one-class SVM only 0 is supported

model_file is the model file generated by svm-train.
test_file is the test data you want to predict.
svm-predict will produce output in the output_file.
```

** `svm-scale` Usage

```
Usage: svm-scale [options] data_filename
options:
-l lower : x scaling lower limit (default -1)
-u upper : x scaling upper limit (default +1)
-y y_lower y_upper : y scaling limits (default: no y scaling)
-s save_filename : save scaling parameters to save_filename
-r restore_filename : restore scaling parameters from restore_filename
```

See 'Examples' in this file for examples.

** Tips on Practical Use

* Scale your data. For example, scale each attribute to [0,1] or [-1,+1].
* For C-SVC, consider using the model selection tool in the tools directory.
* nu in nu-SVC/one-class-SVM/nu-SVR approximates the fraction of training
  errors and support vectors.
* If data for classification are unbalanced (e.g. many positive and
  few negative), try different penalty parameters C by -wi (see
  examples below).
* Specify larger cache size (i.e., larger -m) for huge problems.

** Examples

```
> svm-scale -l -1 -u 1 -s range train > train.scale
> svm-scale -r range test > test.scale
```

Scale each feature of the training data to be in [-1,1]. Scaling
factors are stored in the file range and then used for scaling the
test data.

```
> svm-train -s 0 -c 5 -t 2 -g 0.5 -e 0.1 data_file 
```

Train a classifier with RBF kernel exp(-0.5|u-v|^2), C=10, and
stopping tolerance 0.1.

```
> svm-train -s 3 -p 0.1 -t 0 data_file
```

Solve SVM regression with linear kernel u'v and epsilon=0.1
in the loss function.

```
> svm-train -c 10 -w1 1 -w-2 5 -w4 2 data_file
```

Train a classifier with penalty 10 = 1 * 10 for class 1, penalty 50 =
5 * 10 for class -2, and penalty 20 = 2 * 10 for class 4.

```
> svm-train -s 0 -c 100 -g 0.1 -v 5 data_file
```

Do five-fold cross validation for the classifier using
the parameters C = 100 and gamma = 0.1

```
> svm-train -s 0 -b 1 data_file
> svm-predict -b 1 test_file data_file.model output_file
```

Obtain a model with probability information and predict test data with
probability estimates