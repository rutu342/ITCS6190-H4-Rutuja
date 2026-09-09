# Hands-on L4 — Report

**Name:** Rutuja Hemant Shastri
**Student ID:** 801484366
**Email:** [rshastri@charlotte.edu](mailto:rshastri@charlotte.edu)

---

## What I ran

I followed the steps provided in the README to run the Hadoop MapReduce WordCount program.

First, I started the Hadoop cluster using Docker Compose:

```bash
docker compose up -d
```

I verified that the Hadoop containers were running using:

```bash
docker ps
```

The cluster had the NameNode, ResourceManager, three DataNodes, two NodeManagers, and the HistoryServer running.

I then built the project using Maven:

```bash
mvn clean package
```

The build completed successfully and generated the JAR file:

```text
target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar
```

Next, I copied the JAR file and my input dataset into the ResourceManager container:

```bash
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/tmp/

docker cp shared-folder/input/data/input.txt resourcemanager:/tmp/
```

I opened a shell in the ResourceManager container and moved to `/tmp`:

```bash
docker exec -it resourcemanager bash

cd /tmp
```

I created the input directory in HDFS and uploaded the input file:

```bash
hadoop fs -mkdir -p /input/data

hadoop fs -put ./input.txt /input/data

hadoop fs -ls /input/data
```

The input file was successfully stored in HDFS at `/input/data/input.txt`.

I then ran the MapReduce job:

```bash
hadoop jar /tmp/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar \
  com.example.controller.Controller /input/data/input.txt /output
```

After the job completed, I displayed the results:

```bash
hadoop fs -cat /output/*
```

I then copied the output from HDFS back to my local repository:

```bash
hdfs dfs -get /output /tmp/

exit

docker cp resourcemanager:/tmp/output/. shared-folder/output/
```

After confirming that the output was copied successfully, I stopped the Hadoop cluster:

```bash
docker compose down
```


### My input dataset

I used a dataset about cloud computing and DevOps. The dataset contains three paragraphs describing cloud computing, DevOps practices, automation, infrastructure, containers, monitoring, and software deployment.

The exact contents of my `input.txt` file when I ran the MapReduce job were:

```text
# Create your own input dataset
Cloud computing has changed the way software is developed and deployed. Cloud platforms provide computing resources, storage, networking, and other services that can be accessed on demand. Developers can use cloud resources to build applications without managing physical infrastructure.

DevOps brings development and operations teams together to improve the software development process. DevOps practices encourage automation, continuous integration, continuous delivery, monitoring, and collaboration. Automation helps teams deploy software faster and reduces repetitive manual work.

Cloud computing and DevOps work well together because cloud platforms make it easier to automate infrastructure and application deployment. Containers, orchestration, monitoring, and automated pipelines are commonly used in modern cloud environments. A good DevOps process helps teams deliver reliable software while using cloud resources efficiently.
```

### MapReduce output

The WordCount MapReduce job produced the following output:

```text
and     8
DevOps  4
cloud   4
software        4
teams   3
Cloud   3
computing       3
together        2
monitoring,     2
continuous      2
can     2
platforms       2
development     2
resources       2
the     2
helps   2
delivery,       1
good    1
without 1
deployment.     1
physical        1
operations      1
well    1
storage,        1
encourage       1
orchestration,  1
infrastructure. 1
networking,     1
because 1
process 1
developed       1
while   1
automate        1
services        1
other   1
work.   1
easier  1
efficiently.   1
make    1
dataset 1
are     1
environments.  1
Automation      1
automated       1
resources,      1
deliver 1
faster  1
way     1
accessed        1
modern  1
infrastructure 1
reduces 1
work    1
commonly        1
input   1
build   1
repetitive      1
Containers,     1
automation,     1
use     1
deploy  1
own     1
Developers      1
reliable        1
collaboration.  1
that    1
Create  1
deployed.       1
your    1
demand. 1
manual  1
provide 1
practices       1
improve 1
using   1
application     1
applications    1
brings  1
has     1
process.        1
used    1
integration,    1
pipelines       1
managing        1
changed 1
```

---

## What I observed

The Hadoop cluster started successfully with three DataNodes running. The input file was successfully uploaded to HDFS before the MapReduce job was submitted.

The MapReduce job used one map task and one reduce task. At the beginning of the job, both the map and reduce phases were at 0%. The map phase then reached 100% while the reduce phase was still at 0%. After that, the reduce phase reached 100%, and the job completed successfully.

The job processed 6 input records. The mapper generated 116 intermediate output records. The combiner reduced the 116 records to 85 records before the shuffle and reduce stages. The reducer then produced 85 final output records.

The job completed with zero failed shuffles.

The word counts showed that some words occurred multiple times in the input dataset. For example, `and` occurred 8 times, while `DevOps`, `cloud`, and `software` each occurred 4 times. The words `teams`, `Cloud`, and `computing` also appeared multiple times.

I also noticed that capitalization and punctuation affected the output. For example, `Cloud` and `cloud` were counted as separate words. Similarly, words such as `monitoring,`, `storage,`, and `deployment.` included their punctuation in the output.

The original placeholder line `# Create your own input dataset` was still present in the input file when I ran the job. As a result, words from that line such as `Create`, `your`, `own`, and `dataset` also appeared in the word-count output.

---

## Problems and fixes

During the process, I accidentally entered the HDFS path:

```text
/input/data/input.txt
```

directly into the container shell. This produced the following error:

```text
bash: /input/data/input.txt: No such file or directory
```

The issue was that `/input/data/input.txt` is an HDFS path and is not a command that can be executed directly from the shell.

I verified that the file was correctly stored in HDFS using:

```bash
hadoop fs -ls /input/data
```

The file was present at `/input/data/input.txt`, so I continued with the MapReduce job using the correct HDFS input path.

The MapReduce job completed successfully with one map task, one reduce task, and no failed shuffles.
