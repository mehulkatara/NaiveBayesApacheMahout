Naive Bayes Using Apache Mahout
========================================================
autosize: true
<h3>Fingertis</h3>
<h3>Mehul Katara & Sahil Desai</h3>
<h3>2021-01-18</h3>

What is Naive Bayes ?
========================================================

[Learn Naive Bayes Algorithm](https://www.analyticsvidhya.com/blog/2017/09/naive-bayes-explained/)

[Mahout Naive Bayes Intro](https://mahout.apache.org/docs/latest/tutorials/samsara/spark-naive-bayes.html)

- Mahout currently has two flavors of Naive Bayes.
  + The first is standard  <b>Multinomial Naive Bayes</b>.
  + The second is an implementation of Transformed Weight-normalized <b>Complement Naive Bayes</b>.

***
When one class has more training examples than an other, <b>Naive Bayes</b> selects poor weights for the decision boundary. This is due to under-studied bias effect that shrinks weights for classes with few training examples. To balance the amount of training examples used per estimate, we introduce a <b>"complement class" formulation of Naive Bayes</b>.


Steps to be followed in Mahout
========================================================

1. Getting the data
2. Copping text files to the HDFD
3. Convert our dataset into SequenceFiles
4. Convert SequenceFiles to sparse vector file format
5. Split dataset in Testing and Training
6. Training the model with training dataset
7. Test the model with the testing dataset

Steps 1 : Getting the 20 Newsgroups dataset
========================================================

- The 20 Newsgroups data set is a collection of approximately 20,000 newsgroup documents, partitioned (nearly) evenly across 20 different newsgroups.
<http://qwone.com/~jason/20Newsgroups/>

Download Dataset

<http://qwone.com/~jason/20Newsgroups/20news-bydate.tar.gz>

Steps 2 : Copping text files to the HDFD
========================================================

After downloading our text collections locally, and in order to be able to handle it with Mahout. It's time to copy it to out HDFS.
```
hadoop fs -mkdir 20newsdata

hadoop fs -copyFromLocal 20news-bydate-train/ 20newsdata
```

Steps 3 : Convert our dataset into SequenceFiles
========================================================

- SequenceFile is a hadoop flat file format where each document is represented as a key-value pair.
- Actually, Key is the document id and value is its content

```
mahout seqdirectory -i 20newsdata/20news-bydate-train -o 20newsdataseq-out

-i : specifying the input directory
-o : specifying the output directory
```

Steps 4 : Convert SequenceFiles to sparse vector file format
========================================================

- In order to be able to run properly, most algorithms in text mining require a numerical representation of texts. That's why, we should turn the collections of texts we had in the previous steps into numerical feature vectors. Therefore, every document is represented as a vector where each element of the vector is a word and its weight respectively 

```
mahout seq2sparse -i 20newsdataseq-out/part-m-00000 -o 20newsdatavec -lnorm -nv -wt tfidf 

-i : specifying the input directory
-o : specifying the output directory
-lnorm : output vector to be log normalized
-nv : named vectors
-wt : weight to use here we use tfidf
```

Steps 5 : Split dataset in Testing and Training
========================================================

- Split our dataset into testing and training so we can test our model how accurate it is.

```
mahout split -i 20newsdatavec/tfidf-vectors --trainingOutput 20newsdatatrain --testOutput 20newsdatatest --randomSelectionPct 70 --overwrite --sequenceFiles -xm sequential  

--trainingOutput : training output directory
--testOutput : testing output directory
--randomSelectionPct : split data in percentage for training
--overwrite : overwrite if folder exist
--sequenceFiles : sequential
--xm : execution method sequential or mapreduce
```

Steps 6 : Training the model
========================================================
- Train the model with training dataset

```
mahout trainnb -i 20newsdatatrain -o model -li lableindex -ow -c  

-i : specifying the input directory
-o : specifying the output directory
-li :  specifying the lableindex directory
-ow : overwrite
-c : Complement Naive Bayes
```

Steps 7 : Test the model
========================================================
- Now, Test the model with our testing dataset and check the result

```
mahout testnb -i 20newsdatatest -m model -l lableindex -ow -o results  

-i : specifying the input director
-m : specifying the model
-o : specifying the output directory
-li :  specifying the lableindex directory
-ow : overwrite
```
