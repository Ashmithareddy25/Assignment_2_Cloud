# 📑 Assignment 2: Document Similarity using MapReduce

**Name: Ashmitha Reddy Thota**  
**Student ID: 801400541**  

---

## 🔹 Approach and Implementation

### Mapper Design
Explain the logic of your `Mapper` class:  
- **Input:** Key-value pair received by the Mapper (e.g., line offset, line of text).  
- **Processing:** The Mapper tokenizes each document into words, forms word sets, and generates candidate document pairs.  
- **Output:** Emits intermediate key-value pairs where the key is a document pair `(DocA, DocB)` and the value represents their word sets or common words.  

This helps in preparing data for computing the Jaccard similarity between documents.

---

### Reducer Design
Explain the logic of your `Reducer` class:  
- **Input:** Key is a document pair `(DocA, DocB)` and values are collections of words contributed by the Mapper.  
- **Processing:** The Reducer aggregates word sets, computes the intersection and union of words, and calculates the **Jaccard Similarity = |Intersection| / |Union|**.  
- **Output:** Emits the document pair with their similarity score.  

---

### Overall Data Flow
1. **Input Files** → Split into records and fed into the **Mapper**.  
2. **Mapper Phase** → Produces `(DocA, DocB)` as keys with words as values.  
3. **Shuffle & Sort Phase** → Groups values by document pairs.  
4. **Reducer Phase** → Computes Jaccard similarity for each document pair.  
5. **Final Output** → List of document pairs with similarity scores.  

---

## ⚙️ Setup and Execution

> **Note:** These are the actual commands I used in my assignment.

### 1. Start the Hadoop Cluster
```bash
docker compose up -d
```
### 2. Build the Code
```bash
mvn clean install

### 3. Copy JAR to Docker Container
```bash
docker cp /workspaces/Assignment2-Document-Similarity-using-MapReduce/target/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```
### 4. Move Dataset to Docker Container
```bash
docker cp shared_folder/input_files/ resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```
### 5. Connect to Docker Container
```bash
docker exec -it resourcemanager /bin/bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```
### 6. Set Up HDFS
```bash
hadoop fs -mkdir -p /input/dataset
hadoop fs -put ./input_files /input/dataset
```
### 7. Execute the MapReduce Job
```bash
hadoop jar DocumentSimilarity-0.0.1-SNAPSHOT.jar com.example.controller.DocumentSimilarityDriver /input/dataset/datasets /output
```
### 8. View the Output
```bash
hadoop fs -cat /output/*
```
### 9. Copy Output from HDFS to Local
```bash
hdfs dfs -get /output /opt/hadoop-3.2.1/share/hadoop/mapreduce/
exit
docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output/ output/
```
### 10. Shut Down the Cluster
```bash
docker-compose down -v --remove-orphans
```
## ⚡ Challenges and Solutions

- **Challenge:** Generating unique document pairs without duplicates.  
  **Solution:** Normalized document pairs as `(DocA, DocB)` with lexicographic ordering to avoid `(DocB, DocA)` repetition.  

- **Challenge:** Debugging MapReduce code inside Hadoop Docker containers.  
  **Solution:** Used commands like `hadoop fs -ls`, `hadoop fs -cat`, and `yarn logs -applicationId <id>` to trace issues and verify file paths.  

- **Challenge:** Output directory already exists error when re-running jobs.  
  **Solution:** Changed output path to `/output1`, `/output2`, etc., for subsequent runs or deleted the old HDFS output directory before execution.  

- **Challenge:** Handling tokenization of words across different documents.  
  **Solution:** Applied simple string split logic and filtering to ensure consistent word sets for Jaccard similarity computation.  

---
## 📂 Sample Input

**Input (small_dataset.txt):**
```bash

Document1 this is a sample document
Document2 another document with some words
Document3 more text to test similarity
```

## 📊 Sample Output

**Expected Output:**

```bash

(ds2.txt, ds1.txt)	-> 0.03
(ds3.txt, ds1.txt)	-> 0.01
(ds3.txt, ds2.txt)	-> 0.29
```



