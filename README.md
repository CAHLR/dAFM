# dAFM and its variants

This is a python implementation of the dAFM paper:

Pardos, Z.A., Dadu, A. (2018) dAFM: Fusing Psychometric and Connectionist Modeling for Q-matrix Refinement. Journal of Educational Data Mining. Vol 10(2), 1-27. [**pdf**](https://jedm.educationaldatamining.org/index.php/JEDM/article/download/314/97)

### This repository contains:
- dAFM: dynamic or deep Additive Factors Model
- Deep Knowledge Tracing
- Additive Factors Model
- Skill Model Generation using clustering on distributed representations

### Dependencies
- python3
- pandas
- keras
- theano
- numpy
- scipy
- sklearn
- gensim
- tensorflow
- matlab.engine (for clustering on matlab)

These dependencies may be installed from the requirements file by running the following command:
```
pip3 install -r requirements.txt
```
Python: Only version 3.5.2 is currently supported 
### Dataset:

Dataset should be tab-separated file. Training and test set will be generated on the basis of user-level split (except when using student information). It should have the following attributes:

- **user_id**: unique id of student or user
- **problem_id**: unique id of problem
- **skill_name**: skill associated with the problem
- **correctness**: first attempt correctness of a particular response
- **section**: section information (set of problems belong to a particular section)
- **unit**: contains set of sections

### How to run
1) Clone the github repository.
```
git clone https://github.com/CAHLR/dAFM.git
```
2) Enter into dAFM directory using following command.
```
cd dAFM
```

3) Install required dependencies with the following command.
```
pip3 install -r requirements.txt
```
4) Execute the ```src/main.py``` script using python after specifying the input values. It will train on training set and evaluate the average root mean square error on validation set. It will also save the model in created Accuracy folder. The example dataset is already included in the datasets folder.
**Note:** The detailed description of the input values is mentioned in later section.
```
python3 src/main.py --dataset Example --dataset_path Example/example.txt --skill_name skill_name --dafm fine-tuned No --save_model True
```
5) It will use the load the saved model and evaluate the rmse on testing set. 
```
python3 src/main.py --dataset Example --dataset_path Example/example.txt --skill_name skill_name --dafm fine-tuned No --load_model True sub --puser orig
```

### Input Parameters
The arguements can be passed to the main script py adding them using the syntax given by `python3 src/main.py --arguement_name1  value arguement_name2 value1 value2`.

**Dataset description:**

Arguement Name | Parameters | Description
:--------------------- | :------------- | :--------
**dataset**| required |Specify name of datasets to save its analysis.
value| user_specified_name | like Example, Geometry, ASSISTments
**dataset_path** | required | Absolute path of the dataset file or name of file if present in datasets folder
value| dataset_path | like Example/example.txt
**user_id**| required | name of column that maps to student id
value| user-column-name| default: user_id
**problem_id**| required|name of column that maps to problem id
value| problem-column-name | default: problem_id
**correctness**| required | name of column that maps to first attempt correctness
value| correct-column-name| default: correctness
**skill_name** | optional (not required if using clustering method) | default:None, name of column that maps to skill
value| skill-column-name| KC(KTracedSkill)
**section**| optional | section information and how it should be added
value1| No | default: using no skill information
&nbsp;| concat | new skill model is defined by concatenating section and skill
&nbsp;| one-hot | section information is added as a one-hot vector
value2| section-column-name| default:None, name of column that maps to section
**unit**| optional | if analysis has to be done on a particular unit of dataset
value1| all | default: analysis on all the units
&nbsp;| unit_name | name on particular unit
value2| unit-column-name | default:None, name of column that maps to unit
**unit_users**| optional | concatenate unit and users to increase number of samples instead of responses to reduce memory (use only when unit column name is mentioned)
value| No | default: No unit user concatenation
&nbsp;| all| concatenation for all units
&nbsp;| unit_name | concatenation on a particular unit

**Test-Train type:**

Arguement Name | Parameters | Description
----------------- | ------------- |--------
**puser**| required | default: whether use the validation set or not
value| sub | train on training set (subtrain) and validate on validation set (subtest), display rmse on validation set
&nbsp;| orig | no validation set involved, train on training set, display rmse on testing set if load_model is True else loss on training set itself.
**item_wise** | | item level split to use student information #TODO

**Skill Type:**

Arguement Name | Parameters | Description
----------------- | ------------- |--------
**representation**| required | representation to be used to create the skill model generated by clustering
value| rnn-(dense / correct / incorrect / correct-incorrect) | using the rnn based dkt and the type of vectors used to represent problems
&nbsp;| w2v-(withCorrectness / withoutCorrectness) | using word2vec and the type of vectors used to represent problems
**w2v_params**| optional | values for skipgram model training
value1 | vector_size | default:100, size of representation vectors
paramter2 | window_size | default:20, window size to be used for skip gram
**rnn_params**| optional | values for dkt model training
paramter| hidden_layer_size | default:100, number of nodes in hidden layer of dkt
**clustering_params**| optional |  values for kmeans clustering, should be used with representation
value1 | half, same, double, integer | no. of clusters (times the number of skills in existing skill model)
value2 | euclidean, cosine | distance measure to be used for clustering cosine clustering can be done only using matlab engine

**Predictive Model:**

Arguement Name | Parameters | Description
----------------- | ------------- |--------
**dafm** | required | To use dAFM model
value1 | dafm-afm | same as AFM
&nbsp;| fine-tuned | two time training model
&nbsp;| round-fine-tuned | rounded off Q-matrix
&nbsp;| kcinitialize | train using Q-matrix initialization (all values at once)
&nbsp;| random-uniform | Q-matrix initialization using uniform distribution
&nbsp;| random-normal | Q-matrix initialization using normal distribution
&nbsp;| qjk-dense | using another layer above the original Q-matrix layer **--dense_size same** should also be passed as value. The weight matrix above qjk-layer is an Identity.
&nbsp;| **Advanced Models** | #TODO
&nbsp;| round-fine-tuned_0.5 | extension of round-fine-tuned threshold can be used to round off Q-matrix
&nbsp;| random-qjk-dense-normal | extension of qjk-dense and weight matrix above qjk layer is initialized to be normally distributed random matrix
&nbsp;| random-qjk-dense-uniform | extension of qjk-dense and weight matrix above qjk layer is initialized to be uniformly distributed random matrix
&nbsp;| random-qjk-dense^0.3_0.9_different | extension of qjk-dense size of Qjk dense is 0.3 times of the 0.9 total skills.
&nbsp;| random-uniform_0.3_different | extension of random-uniform reduces Qjk layer size to 0.3 times the total skills
value2| No | Train or Predict without using batches (masking on complete training set)
&nbsp;| Yes | Train or Predict using different batch sizes  (masking on subset of training set having similar responses)
**dafm_params** | optional |  values for dafm model training
value1| linear | default: No activation
&nbsp;| sigmoid | applies sigmoid activation on Qjk layer
&nbsp;| custom-a | variation of sigmoid in which x is replaced by ax larger a more steeper the curve, a must be integer
value2| rmsprop | default: optimizer to do backpropogation
&nbsp;| adam, adagrad
value3| 0.1 | default: learning rate for traininf
**afm** | required | To use AFM model
value1| afm-keras | applies neural network implementation of afm
&nbsp;| afm-liblinear | applies logistic regression implementation of afm
**dkt**| required | To use DKT model
value| None | default: do not use dkt
&nbsp;| dkt | use dkt model and results will be saved with filenames having dkt
**theta**| | to include student information
&nbsp;| | #ToDo

**Save and Load:**

Arguement Name | Parameters | Description
----------------- | ------------- |--------
**save_model**| optional | save the trained model
value | False/True | True to save the model
**load_model**| optional | to load the trained model
value1 |False/True | True to load the saved model
value2 | sub | load the model that is trained on subtrain
&nbsp;| orig | load the model that is train on train (use only when puser is orig)
**skill_wise**| optional | save the modified Q-matrix and its interpretation plot
&nbsp;| | #ToDo


### Directory Structure:

```
├── Accuracy
│   └── Example
│       ├── orig@all
│       │   ├── dafm-afm$no$linear$rmsprop$0.1.acc
│       │   ├── Losses
│       │   │   └── dafm-afm$no$linear$rmsprop$0.1.csv
│       │   ├── Model
│       │   │   └── dafm-afm$no$linear$rmsprop$0.1.h5
│       │   └── Prediction
│       │       └── dafm-afm$no$linear$rmsprop$0.1.acc
│       └── sub@all
│           ├── Losses
│           │   ├── dafm-afm$no$linear$rmsprop$0.1.csv
│           │   ├── dafm-afm$no$linear$rmsprop$0.1_rnn-correct_100_4_euclidean.csv
│           │   └── dafm-afm$no$linear$rmsprop$0.1_rnn-correct_100_same_euclidean.csv
│           └── Prediction
│               ├── dafm-afm$no$linear$rmsprop$0.1.acc
│               ├── dafm-afm$no$linear$rmsprop$0.1_rnn-correct_100_4_euclidean.acc
│               └── dafm-afm$no$linear$rmsprop$0.1_rnn-correct_100_same_euclidean.acc
├── AFM
│   ├── afm_keras.py
│   ├── afm_liblinear.py
│   └── load_data.py
├── DAFM
│   ├── dafm.py
│   └── load_data.py
├── data
│   ├── load_data.py
│   └── splitData.py
├── datasets
│   └── Example
│       ├── example.txt
│       └── Users
│           ├── subtest.csv
│           ├── subtrain.csv
│           ├── test.csv
│           └── train.csv
├── log
│   ├── 3.mat
│   └── 3.pro
├── Qmatrix
│   └── qmatrix.py
├── README.md
├── Representation
│   ├── dkt.py
│   ├── problem2vec.py
│   └── rnn.py
└── src
    ├── load_data.py
    └── main.py
```
#ToDo Add each file description
