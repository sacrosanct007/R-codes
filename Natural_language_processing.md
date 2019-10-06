

Natural Language Processing---------------------------------------------------------------------------------------------------------------------------------


```r
# Import the dataset
dataset_original=read.delim('Restaurant_Reviews.tsv',quote='',stringsAsFactors = FALSE)
#head(dataset)

# Cleaning the text
#install.packages('tm')
#install.packages('SnowballC')
library(tm)
```

```
## Loading required package: NLP
```

```r
corpus=VCorpus(VectorSource(dataset_original$Review))
as.character(corpus[[1]])
```

```
## [1] "Wow... Loved this place."
```

```r
as.character(corpus[[841]])
```

```
## [1] "for 40 bucks a head, i really expect better food."
```

```r
corpus=tm_map(corpus,content_transformer(tolower))
corpus=tm_map(corpus, removeNumbers)
corpus=tm_map(corpus, removePunctuation)
corpus=tm_map(corpus, removeWords,stopwords())
corpus=tm_map(corpus, stemDocument)
corpus=tm_map(corpus, stripWhitespace)
as.character(corpus[[1]])
```

```
## [1] "wow love place"
```

```r
as.character(corpus[[841]])
```

```
## [1] "buck head realli expect better food"
```

```r
# Creating the Bag of words model
dtm=DocumentTermMatrix(corpus)
dtm=removeSparseTerms(dtm,0.999)
dtm
```

```
## <<DocumentTermMatrix (documents: 1000, terms: 691)>>
## Non-/sparse entries: 4549/686451
## Sparsity           : 99%
## Maximal term length: 12
## Weighting          : term frequency (tf)
```

```r
dataset=as.data.frame(as.matrix(dtm))
dataset$Liked=dataset_original$Liked

# Prefer to develop Random Forest, Decision tree or Naive Bayes model for Natural Language processing
# Below is the Random Forest implementation

# Convert dependent variable into factor typr
dataset$Liked=factor(dataset$Liked,levels=c(0,1))

# Split the dataset into training and test set
library(caTools)
set.seed(123)
Split=sample.split(dataset$Liked,SplitRatio = 0.8)
training_set=subset(dataset,Split==TRUE)
test_set=subset(dataset,Split==FALSE)

# Feature Scaling- not required here for NLP as our variables are already 0 or 1
# training_set[,1:2]=scale(training_set[,1:2])
# test_set[,1:2]=scale(test_set[,1:2])

# Fit model to the dataset
#install.packages('randomForest')
library(randomForest)
```

```
## randomForest 4.6-14
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```r
RF_classifier=randomForest(x=training_set[,-692],y=training_set[,692],ntree=10)

#summary(RF_classifier)

# Predict the test set result
y_pred=predict(RF_classifier,newdata=test_set[,-692])


# making the confusion matrix
cm=table(test_set[,692],y_pred)
cm
```

```
##    y_pred
##      0  1
##   0 82 18
##   1 23 77
```

