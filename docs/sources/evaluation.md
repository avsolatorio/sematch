## Word Similarity Evaluation

We have collected some well known word similarity datasets for evaluating semantic similarity metrics. Several python classes can be used to prepare the dataset and evaluate the metric automatically. The word similarity datasets included in Sematch are listed as below:

- [Rubenstein and Goodenough (RG)](http://www.cs.cmu.edu/~mfaruqui/word-sim/EN-RG-65.txt) 

Herbert Rubenstein and John B. Goodenough. 1965. Contextual correlates of synonymy. Commun. ACM 8, 10 (October 1965), 627-633. DOI=10.1145/365628.365657 

- [Miller and Charles (MC)](http://www.cs.cmu.edu/~mfaruqui/word-sim/EN-MC-30.txt) 

Miller, George A., and Walter G. Charles. "Contextual correlates of semantic similarity." Language and cognitive processes 6.1 (1991): 1-28.

- [Wordsim353 (WS353)](http://www.cs.technion.ac.il/~gabr/resources/data/wordsim353/) 

Lev Finkelstein, Evgeniy Gabrilovich, Yossi Matias, Ehud Rivlin, Zach Solan, Gadi Wolfman, and Eytan Ruppin, "Placing Search in Context: The Concept Revisited", ACM Transactions on Information Systems, 20(1):116-131, January 2002 

- [wordsim353 similarity and relatedness (WS353Sim)](http://alfonseca.org/eng/research/wordsim353.html) 

Eneko Agirre, Enrique Alfonseca, Keith Hall, Jana Kravalova, Marius Pasca, Aitor Soroa, A Study on Similarity and Relatedness Using Distributional and WordNet-based Approaches, In Proceedings of NAACL-HLT 2009.

- [SimLex-999 (SIMLEX)](http://www.cl.cam.ac.uk/~fh295/simlex.html) 

SimLex-999: Evaluating Semantic Models with (Genuine) Similarity Estimation. 2014. Felix Hill, Roi Reichart and Anna Korhonen. Preprint pubslished on arXiv. arXiv:1408.3456

- [Multilingual Word Similarity](http://lcl.uniroma1.it/similarity-datasets/)

Camacho-Collados, José, Mohammad Taher Pilehvar, and Roberto Navigli. "A Framework for the Construction of Monolingual and Cross-lingual Word Similarity Datasets." ACL (2). 2015.


When developing new similarity metrics, proper evaluation is important, whereas sometimes it is tedious. Sematch helps to save such efforts by providing a evaluation framework, where similarity metrics are evaluated with common word similarity datasets and can be compared with other similarity metrics. Futhermore, the Steiger's Z Significance Test is included to perform dependent statistical significance test. For example, we have developed a novel similarity metric WPath, and we want to evaluate with some datasets and compare it to Lin method.

```python
from sematch.evaluation import WordSimEvaluation
from sematch.semantic.similarity import WordNetSimilarity
evaluation = WordSimEvaluation()
print evaluation.dataset_names()
wns = WordNetSimilarity()
#define similarity metrics
lin = lambda x, y: wns.word_similarity(x, y, 'lin')
wpath = lambda x, y: wns.word_similarity_wpath(x, y, 0.8)
#evaluate similarity metrics
print evaluation.evaluate_metric('wpath', wpath, 'noun_simlex')
#performa Steiger's Z significance Test
print evaluation.statistical_test('wpath', 'path', 'noun_simlex')
#define multilingual similarity metrics Spanish-Spanish 
wpath_es = lambda x, y: wns.monol_word_similarity(x, y, 'spa', 'path')
#English-Spanish
wpath_en_es = lambda x, y: wns.crossl_word_similarity(x, y, 'eng', 'spa', 'wpath')
#Evaluate multilingual word similarity datasets
print evaluation.evaluate_metric('wpath_es', wpath_es, 'rg65_spanish')
print evaluation.evaluate_metric('wpath_en_es', wpath_en_es, 'rg65_EN-ES')

```

## Category Classification Evaluation

Although the word similarity correlation measure is the standard way to evaluate the semantic similarity metrics, it relies on human judgements over word pairs which may not have same performance in real applications. Therefore, apart from word similarity evaluation, the Sematch evaluation framework also includes a simple aspect category classification for Aspect Based Sentiment Analysis. We use the dataset from SemEval2015 and SemEval2016, sentence-level Aspect-based Sentiment Analysis. The original dataset can be found in [Aspect Based Sentiment Analysis 15](http://alt.qcri.org/semeval2015/task5/) and [Aspect Based Sentiment Analysis 16](http://alt.qcri.org/semeval2016/task5/).

To evaluate the mode, you need to first define a word similarity measurement function, and then train and evaluate the classification model.

```python
from sematch.evaluation import AspectEvaluation
from sematch.application import SimClassifier, SimSVMClassifier
from sematch.semantic.similarity import WordNetSimilarity

# create aspect classification evaluation
evaluation = AspectEvaluation()

# load the dataset
X, y = evaluation.load_dataset()

# define word similarity function
wns = WordNetSimilarity()
word_sim = lambda x, y: wns.word_similarity(x, y)

# Train and evaluate metrics with unsupervised classification model
simclassifier = SimClassifier.train(zip(X,y), word_sim)
evaluation.evaluate(X,y, simclassifier)

-----Evaluation Results-----
macro averge:  (0.65319812882333839, 0.7101245049198579, 0.66317566364913016, None)
micro average:  (0.79210167952791644, 0.79210167952791644, 0.79210167952791644, None)
weighted average:  (0.80842645056024054, 0.79210167952791644, 0.79639496616636352, None)
accuracy:  0.792101679528
             precision    recall  f1-score   support

    SERVICE       0.50      0.43      0.46       519
 RESTAURANT       0.81      0.66      0.73       228
       FOOD       0.95      0.87      0.91      2256
   LOCATION       0.26      0.67      0.37        54
   AMBIENCE       0.60      0.70      0.65       597
     DRINKS       0.81      0.93      0.87       752

avg / total       0.81      0.79      0.80      4406

           |                        R      |
           |                        E      |
           |    A              L    S      |
           |    M              O    T    S |
           |    B    D         C    A    E |
           |    I    R         A    U    R |
           |    E    I    F    T    R    V |
           |    N    N    O    I    A    I |
           |    C    K    O    O    N    C |
           |    E    S    D    N    T    E |
-----------+-------------------------------+
  AMBIENCE | <223>   .   13   43  179   61 |
    DRINKS |   15 <151>  54    4    1    3 |
      FOOD |   99   24<1960>  37   76   60 |
  LOCATION |    2    .    3  <36>  13    . |
RESTAURANT |   81   12   29   19 <417>  39 |
   SERVICE |   30    .   10    2    7 <703>|
-----------+-------------------------------+
(row = reference; col = test)


# Train and evaluate metrics with supervised classification model
simSVMclassifier = SimSVMClassifier.train(X, y, word_sim) #supervised classification
evaluation.evaluate(X, y, simSVMclassifier)

-----Evaluation Results-----
macro averge:  (0.87738328966718993, 0.80275862524008135, 0.83522698525943129, None)
micro average:  (0.87721289151157511, 0.87721289151157511, 0.87721289151157511, None)
weighted average:  (0.87555902983892719, 0.87721289151157511, 0.87314386708061753, None)
accuracy:  0.877212891512
             precision    recall  f1-score   support

    SERVICE       0.85      0.63      0.72       519
 RESTAURANT       0.92      0.85      0.89       228
       FOOD       0.89      0.97      0.93      2256
   LOCATION       0.91      0.74      0.82        54
   AMBIENCE       0.75      0.71      0.73       597
     DRINKS       0.94      0.91      0.93       752

avg / total       0.88      0.88      0.87      4406

           |                        R      |
           |                        E      |
           |    A              L    S      |
           |    M              O    T    S |
           |    B    D         C    A    E |
           |    I    R         A    U    R |
           |    E    I    F    T    R    V |
           |    N    N    O    I    A    I |
           |    C    K    O    O    N    C |
           |    E    S    D    N    T    E |
-----------+-------------------------------+
  AMBIENCE | <328>   4   58    .  111   18 |
    DRINKS |    3 <194>  29    .    .    2 |
      FOOD |   22    8<2194>   .   19   13 |
  LOCATION |    3    .    2  <40>   9    . |
RESTAURANT |   21    4  138    4 <422>   8 |
   SERVICE |    9    .   56    .    . <687>|
-----------+-------------------------------+
(row = reference; col = test)
```

