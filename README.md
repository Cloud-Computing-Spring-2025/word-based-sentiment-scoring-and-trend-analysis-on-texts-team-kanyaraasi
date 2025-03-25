# multi-stage-sentiment-analysis-Mapreduce_hive
This is Multi-Stage Sentiment Analysis on Historical Literature application is developed with map reduce and hive

---

## 🎯 Objectives

- Clean and standardize raw text from books
- Extract metadata (Book ID, Title, Year)
- Perform word frequency analysis with lemmatization
- Assign sentiment scores using AFINN lexicon
- Analyze trends by decade
- Extract and analyze bigrams using Hive UDF

---

## 🛠️ Technologies Used

- Java (JDK 8+)
- Hadoop MapReduce (2.7.4)
- Hive (with custom Java UDF)
- Stanford CoreNLP (for lemmatization)
- AFINN-111.txt (Sentiment Lexicon)
- GitHub for version control

---

## 📦 Dependencies

- Java
- Hadoop
- Hive
- CoreNLP (add to classpath)
- AFINN-111.txt file for sentiment scores

---

## 🚀 How to Run

Each task is implemented as a MapReduce job with a separate driver.

## 🔹 Task 1: Preprocessing

#### Objective: 
Extract metadata and clean text for further analysis.
#### Mapper:
•	Reads each line of the book.

•	Accumulates full content.

•	Extracts metadata: book ID (from file name), title, and release year (from header).

•	Cleans text: converts to lowercase, removes punctuation, filters stop words.

•	Emits a composite key: bookID_year_title and cleaned text as value.
#### Reducer:
•	Concatenates all lines (if multiple for a book).

•	Outputs key-value with full preprocessed content.

### Code explanation:

### PreprocessingMapper.java
#### Key Steps:
#### setup():

Loads stop words into a HashSet.

Retrieves the filename for metadata extraction.

#### map():

Appends each line to a buffer to process the full file in cleanup().

#### cleanup():

-Extracts metadata:

Title: line → title

Release date: line → year

pgXXXX.txt filename → book ID

Cleans the text (lowercase, remove punctuation, remove stopwords).

### PreprocessingReducer.java
#### Function: 
Combines cleaned text from the same book (in case of splits).
#### Key Step:
-Appends multiple mapper outputs (if any) for the same key.

-Writes out the cleaned, ready-to-process text.

## 🔹 Task 2: Word Frequency Analysis with Lemmatization

#### Objective: 
Tokenize text and compute lemma frequency per book and year.
#### Mapper:

•	Splits lines using tab into metadata and content.

•	Uses Stanford CoreNLP Simple API for lemmatization.

•	Emits: bookID, lemma, year with value 1.

#### Reducer:
•	Aggregates frequency of each lemma for a book in a given year.

### Code explanation:

### WordFrequencyMapper.java
#### Function:
Lemmatizes text and emits word counts.

### Key Steps:
Splits input into:
header = bookID_year_title
text = cleaned content
Uses Stanford CoreNLP's Sentence.lemmas() to lemmatize text.

Emits:
key = bookID, lemma, year
value = 1

### WordFrequencyReducer.java
#### Function:
Sums the counts of each lemma.

Straightforward aggregation using a counter.

Emits total frequency for each (bookID, lemma, year).


## 🔹 Task 3: Sentiment Scoring

#### Objective: 
Compute total sentiment score for each book using AFINN lexicon.

#### Mapper:
•	Loads AFINN-111 sentiment lexicon during setup.

•	Tokenizes each word and checks for sentiment score.

•	Emits: (bookID, year) and sentiment score.

#### Reducer:
•	Aggregates sentiment score for each book.

### Code explanation:

### SentimentScoreMapper.java
#### Function:
Assigns sentiment scores using AFINN lexicon.

#### Key Steps:
-In setup():

    Loads the AFINN file into a HashMap<String, Integer>.

-In map():

    Tokenizes cleaned text.

    If a token exists in the lexicon

### SentimentScoreReducer.java
#### Function: 
Totals sentiment scores per book and year.


## 🔹 Task 4: Trend Analysis & Aggregation

#### Objective: 
Aggregate scores by decade to observe long-term trends.

#### Mapper:
•	Extracts decade from year.

•	Emits (bookID, decade) and score.

#### Reducer:

•	Sums scores per decade.

### Code explanation:

### TrendMapper.java
#### Function: 
Converts year → decade and prepares data for aggregation.

#### Key Steps:
 -Input: "bookID, year\t sentimentScore"

 -Converts year like 1994 → 1990s

### TrendReducer.java
Sums up all sentiment scores per (bookID, decade).

## Steps to run

Commands to run

```bash 
docker compose up -d
```
Generate jar files
```bash 
mvn clean install
```

### Task 1

Transfer jar file to docker container
```bash 
docker cp target/PreprocessingJob.jar resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
Transfer input files to docker container:
```bash 
docker cp shared-folder/input/pg8183.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg8395.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg10148.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg10662.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg16767.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg36383.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg66112.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg69543.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg72945.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```
```bash 
docker cp shared-folder/input/pg73936.txt resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```

commands to run in hadoop container

```bash
docker exec -it resourcemanager /bin/bash
```

```bash
cd /opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
hadoop fs -mkdir -p /input/dataset
```
```bash
hadoop fs -put ./pg8183.txt /input/dataset
```
```bash
hadoop fs -put ./pg8395.txt /input/dataset
```
```bash
hadoop fs -put ./pg10148.txt /input/dataset
```
```bash
hadoop fs -put ./pg10662.txt /input/dataset
```
```bash
hadoop fs -put ./pg16767.txt /input/dataset
```
```bash
hadoop fs -put ./pg36383.txt /input/dataset
```
```bash
hadoop fs -put ./pg66112.txt /input/dataset
```
```bash
hadoop fs -put ./pg69543.txt /input/dataset
```
```bash
hadoop fs -put ./pg72945.txt /input/dataset
```
```bash
hadoop fs -put ./pg73936.txt /input/dataset
```
```bash
hadoop jar PreprocessingJob.jar /input/dataset /output
```
```bash
hdfs dfs -get /output /opt/hadoop-2.7.4/share/hadoop/mapreduce/
```
```bash
exit
```
```bash
docker cp resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce/output/ shared-folder/output/
```

### Task 2

Transfer jar file to docker container
```bash 
docker cp target/WordFrequencyJob.jar resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
docker exec -it resourcemanager /bin/bash
```

```bash
cd /opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
hadoop jar WordFrequencyJob.jar /output/part-r-00000 /output/task2
```
```bash
hdfs dfs -get /output/task2 /opt/hadoop-2.7.4/share/hadoop/mapreduce/
```
```bash
exit
```
```bash
docker cp resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce/task2 shared-folder/output/task2
```

### Task 3

Transfer jar file to docker container
```bash 
docker cp target/SentimentScoringJob.jar resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
docker exec -it resourcemanager /bin/bash
```

```bash
cd /opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
hadoop jar SentimentScoringJob.jar /output/task2/part-r-00000 /output/task3
```
```bash
hdfs dfs -get /output/task3 /opt/hadoop-2.7.4/share/hadoop/mapreduce/
```
```bash
exit
```
```bash
docker cp resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce/task3 shared-folder/output/task3
```

### Task 4

Transfer jar file to docker container
```bash 
docker cp target/TrendAggregationJob.jar resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
docker exec -it resourcemanager /bin/bash
```

```bash
cd /opt/hadoop-2.7.4/share/hadoop/mapreduce
```

```bash
hadoop jar TrendAggregationJob.jar /output/task3/part-r-00000 /output/task4
```
```bash
hdfs dfs -get /output/task4 /opt/hadoop-2.7.4/share/hadoop/mapreduce/
```
```bash
exit
```
```bash
docker cp resourcemanager:/opt/hadoop-2.7.4/share/hadoop/mapreduce/task4 shared-folder/output/task4
```
## Challenges Faced
- Integration of CoreNLP with Hadoop MapReduce.
- Managing memory during lexicon loading.
- Handling numeric or unrelated tokens.

