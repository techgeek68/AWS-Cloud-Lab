# Implementing the MapReduce Programming Model Using Amazon EMR

---

**Objectives**
  - Understand the MapReduce programming model and its phases
  - Set up an Amazon EMR cluster with Hadoop
  - Execute a word count MapReduce job
  - Analyze MapReduce job execution and parallel efficiency
  - Compare MapReduce with traditional sequential processing

---

**Theory**

MapReduce is a programming model for processing large datasets in parallel across a distributed cluster. It works in three phases:

**Map Phase:** 
  - Input data is split into chunks. Each mapper processes a chunk independently and produces intermediate key value pairs.

**Shuffle and Sort Phase:** 
  - The intermediate key value pairs are grouped by key and forwarded to the appropriate reducers.

**Reduce Phase:** 
  - Each reducer aggregates all values for a given key and produces the final output.

**Parallel Efficiency:** 
  - MapReduce achieves parallelism by distributing map tasks across nodes in the cluster. Efficiency depends on data locality (processing data where it already resides), even partitioning of input splits, and minimizing the volume of data transferred during the shuffle phase.

**Amazon EMR (Elastic MapReduce)** 
  - Is a managed cluster platform that supports big data frameworks like Hadoop and Spark. It handles provisioning, configuration, and tuning automatically, removing most of the operational overhead involved in running a Hadoop cluster.

**Thread vs. Task vs. MapReduce:**

| Model | Scope | Notes |
|---|---|---|
| Thread Programming | Single machine, shared memory | Requires low level synchronization |
| Task Programming | Single machine, higher level abstraction | Uses schedulers like Java ForkJoin |
| MapReduce | Distributed across machines | Fault tolerant, built for massive datasets |

---

**Procedure**


**Step 1: Create the Input File and Upload to S3**

**1a. Create the S3 Bucket**

  - Go to **AWS Console** -> Search **S3** -> Click **Create bucket**
  - Set **Bucket name:** `emr-lab-<your-name>`       *(replace `<your-name>` with your name)*
  - **Bucket type** -> `General purpose`
  - **ACLs** -> `Disabled`
  - Leave all other settings as default -> Click **Create bucket**
  - Open your bucket -> Click **Create folder** -> Name it `logs` -> Click **Create folder** *(at bucket root)*
  - Click **Create folder** again -> Name it `input` -> Click **Create folder** *(at bucket root)*

>*Sample: [Optional]*

<img width="1457" height="335" alt="Screenshot 2026-04-20 at 7 14 11 AM" src="https://github.com/user-attachments/assets/d7e33f04-7341-43c7-920c-472360170ab9" />


**1b. Create the Input File Locally**

- On your local machine, create `input.txt`:
  - **Linux/Unix:** `vim input.txt`
  - **Windows:** `notepad input.txt`
- Paste the following content:
  ```
  Cloud computing is the future of computing
  Cloud services include IaaS, PaaS, and SaaS
  Virtualization enables cloud computing
  Cloud computing provides scalability and elasticity
  ```
- Save and close the file


**1c. Upload to S3**
  - Open your bucket -> Click into the `input/` folder
  - Click **Upload** -> Click **Add files** -> Select your `input.txt` file
  - Click **Upload**

> *Screenshot checkpoint: S3 bucket showing `input/input.txt` (only `input.txt` inside `input/`, no subfolders)*

>Sample:

<img width="1457" height="330" alt="Screenshot 2026-04-20 at 7 19 09 AM" src="https://github.com/user-attachments/assets/249cdb14-b1db-413b-9641-db09ece73862" />



**Step 2: Create the EMR Cluster**

  1. Go to **AWS Console** -> Search **EMR** -> Click **Create cluster**
  2. Fill in the configuration:

| Setting | Value |
|---|---|
| **Cluster name** | `MapReduce-Lab-Cluster` |
| **EMR Release** | `emr-7.13.0` |
| **Application bundle** | **Core Hadoop** (Hadoop 3.4.2, Hive 3.1.3, Hue 4.11.0, Pig 0.17.0, Tez 0.10.2) |
| **Primary instance type** | `m5.xlarge` |
| **Core instance type** | `m5.xlarge` |
| **Task node** | Click **Remove instance group** to remove Task node |
| **EBS root volume size** | `15 GiB` (default) |
| **Core instance count** | `2` |
| **Cluster configuration** |Uniform instance groups |
| **Cluster scaling** | Set cluster size manually |
| **Networking-VPC** | Leave default selected |
| **Networking-Subnet** | Leave default selected  |
| **Security groups** | Leave as **EMR-managed** for both Primary and Core nodes |
| **Cluster termination** | Automatically terminate cluster after idle time |
| **Cluster termination and node replacement** | Idle time `01:00:00` (1 hour) |
| **Termination protection** | **Off** |
| **EC2 Key pair** | Select your existing key pair from the dropdown |
| **Cluster logs** | Amazon S3 location `s3://emr-lab-<your-id>/logs/` (`Copy S3 uri` of logs folder and paste) |
| **Service role** | `EMR_DefaultRole` |
| **EC2 instance profile** | `EMR_EC2_DefaultRole` |
| **Operating system** | Amazon Linux (default, latest updates applied) |

5. Click **Create cluster**

**Step 3: Wait for the Cluster to Become Ready**

  1. You'll be redirected to the cluster detail page
  2. Watch the **Status** badge cycle through: `Starting` -> `Bootstrapping` -> `Waiting`
  3. This typically takes 8-15 minutes

>*Screenshot checkpoint: Cluster showing status [Mandatory]*

>Sample:

<img width="1455" height="419" alt="Screenshot 2026-04-20 at 12 56 07 PM" src="https://github.com/user-attachments/assets/2484406a-a739-4306-96a1-c88206284e12" />

<img width="1455" height="416" alt="Screenshot 2026-04-20 at 12 58 09 PM" src="https://github.com/user-attachments/assets/7341f377-faf0-4abb-ab39-94fc1ebdad3a" />


**Step 4: Write and Upload the MapReduce Scripts**

**4a. Create the Python Files Locally**

>Linux/Unix: vim mapper.py

>Windows: notepad mapper.py

```python name=mapper.py
#!/usr/bin/env python3
import sys

for line in sys.stdin:
    line = line.strip().lower()
    words = line.split()
    for word in words:
        print(f"{word}\t1")
```
>Linux/Unix: vim reducer.py

>Windows: notepad reducer.py

```python name=reducer.py
#!/usr/bin/env python3
import sys

current_word = None
current_count = 0

for line in sys.stdin:
    line = line.strip()
    word, count = line.split('\t', 1)
    count = int(count)

    if current_word == word:
        current_count += count
    else:
        if current_word:
            print(f"{current_word}\t{current_count}")
        current_word = word
        current_count = count

if current_word:
    print(f"{current_word}\t{current_count}")
```

**4b. Upload Scripts to S3**

  1. Go to your S3 bucket -> Click **Create folder** -> Name it `scripts` -> Click **Create folder**
  2. Click into `scripts/` folder -> Click **Upload**
  3. Click **Add files** -> Select **both** `mapper.py` and `reducer.py`
  4. Click **Upload**

>*Screenshot checkpoint: S3 `scripts/` folder showing both files [Mandatory].*

>Sample:

<img width="1459" height="338" alt="Screenshot 2026-04-20 at 7 22 00 AM" src="https://github.com/user-attachments/assets/edda2dea-cd85-48ca-af7f-1d6e892ff5de" />


**Step 5: Add a Streaming Step to the Cluster**

1. Go to **EMR Console** -> Click your `MapReduce-Lab-Cluster`
2. Click the **Steps** tab -> Click **Add step**
3. In the dialog, configure:

| Field | Value |
|---|---|
| **Step type** | `Streaming program` |
| **Name** | `WordCount-Job` |
| **Mapper** | `s3://emr-lab-<your-id>/scripts/mapper.py` |
| **Reducer** | `s3://emr-lab-<your-id>/scripts/reducer.py` |
| **Input location** | `s3://emr-lab-<your-id>/input/` |
| **Output location** | `s3://emr-lab-<your-id>/output/wordcount/` |
| **Step action Action if step fails** | `Continue` |

> The **output path must NOT already exist** in S3. Hadoop will create it automatically.

4. Click **Add step**

>Sample:

<img width="1454" height="685" alt="Screenshot 2026-04-20 at 1 01 55 PM" src="https://github.com/user-attachments/assets/f9a9eecc-f4b3-43f3-bae2-2d45ba1def53" />


**Step 6: Monitor Job Execution**

  1. Stay on the **Steps** tab of your cluster
  2. Watch the `WordCount-Job` step move through statuses: Pending -> Running -> Completed
  3. Click on the step name to view:
     - **Log files** (stdout, stderr, controller)
     - Direct links to YARN/Hadoop logs

>*Screenshot checkpoint: Step showing Completed status in green*

>Sample:

<img width="1456" height="576" alt="Screenshot 2026-04-20 at 1 08 04 PM" src="https://github.com/user-attachments/assets/32521707-ae3c-44b5-af52-7c4021da591e" />


**Step 7: View the Output in S3**

1. Go to **S3** -> Navigate to:
   ```
   s3://emr-lab-<your-id>/output/wordcount/
   ```
2. You'll see files like `part-00000`, `part-00001`, and `_SUCCESS`
3. Click *part-00000* -> Click Download
4. Open the file; it should look like:

<img width="294" height="309" alt="Screenshot 2026-04-20 at 1 10 52 PM" src="https://github.com/user-attachments/assets/250e0dd6-6c95-402d-bec1-b979cdf6c324" />


>*Screenshot: S3 output folder [Mandatory]*

<img width="1455" height="374" alt="Screenshot 2026-04-20 at 1 12 39 PM" src="https://github.com/user-attachments/assets/fcd7c577-5b58-46da-bfbd-fdfcddc6c1b0" />


**Step 8: Examine Hadoop Job Logs and Metrics**
  - Go to **EMR Console** → Click your cluster
  - Click the **Application user interfaces** tab
  - Under **On-cluster user interfaces**, click **Resource Manager**
  - In the Hadoop Resource Manager UI:
  - Click **Applications** (left menu) -> Click your completed job

>*Screenshot checkpoint: Hadoop ResourceManager job statistics page [Mandatory].*

>Sample:

<img width="1284" height="286" alt="Screenshot 2026-04-20 at 1 32 10 PM" src="https://github.com/user-attachments/assets/5083af90-a379-4966-b05a-f566fb37ddbb" />

[All Applications.pdf](https://github.com/user-attachments/files/26887426/All.Applications.pdf)



**Step 9: Run a Comparison with a Larger Dataset**

**9a. Download a Large Text File**

1. Go to [Project Gutenberg](https://www.gutenberg.org/)
2. Download a plain text book (e.g., *War and Peace* — `.txt` format, ~3MB)

**9b. Upload to S3**

1. Go to your S3 bucket → `input/` folder
2. Upload the large `.txt` file (e.g., `war_and_peace.txt`)

**9c. Run the Same MapReduce Job**

1. In EMR Console → **Steps** tab → **Add step**
2. Use the same mapper/reducer, but update:

| Field | Value |
|---|---|
| **Name** | `WordCount-Large` |
| **Input location** | `s3://emr-lab-<your-id>/input/war_and_peace.txt` |
| **Output location** | `s3://emr-lab-<your-id>/output/wordcount-large/` |

3. Note the execution time from the Steps tab

**9d. Compare Execution Times**

| Job | Input Size | Execution Time |
|---|---|---|
| `WordCount-Job` | ~200 bytes | ~1–2 min |
| `WordCount-Large` | ~3 MB | ~2–4 min |

> **Key insight:** EMR's distributed processing means scaling the cluster (more Core nodes) reduces execution time proportionally for large datasets.

*Screenshot checkpoint: Steps tab showing both completed jobs with their durations*


Step 9: Run a Comparison with a Larger Dataset

- Visit [Project Gutenberg](https://www.gutenberg.org) → Download any public domain book as **Plain Text UTF-8** (e.g., *Moby Dick*)
- Save it as `large_input.txt`
- Go to **S3** → `emr-lab-<your-name>` → `input/` folder
- Click **Upload** → **Add files** → Select `large_input.txt` → Click **Upload**
- Go to **EMR** → your cluster → **Steps** tab → Click **Add step**

| Field | Value |
|---|---|
| **Step type** | `Streaming program` |
| **Name** | `WordCount-Job-Large` |
| **Mapper** | `s3://emr-lab-<your-name>/scripts/mapper.py` |
| **Reducer** | `s3://emr-lab-<your-name>/scripts/reducer.py` |
| **Input location** | `s3://emr-lab-<your-name>/input/large_input.txt` |
| **Output location** | `s3://emr-lab-<your-name>/output/wordcount-large/` |
| **Action on failure** | `Continue` |

- Submit and wait for **Completed**
- Compare **Elapsed time** of both steps

> 📸 **Screenshot checkpoint:** Both steps showing elapsed time for comparison

---

### Step 10: Terminate the EMR Cluster

- Go to **Amazon EMR** → **Clusters**
- Check the checkbox next to `MapReduce-Lab-Cluster`
- Click **Terminate**
- If termination protection is **On** → click **Change** → turn it **Off** → then confirm **Terminate**
- Wait for status to show `Terminated`

> 📸 **Screenshot checkpoint:** Termination confirmation or cluster showing **Terminated** status

> ⚠️ **Always terminate your cluster after the lab to avoid ongoing charges.**

---

**Results**

- An EMR cluster with two core nodes was successfully provisioned.
- The Hadoop Streaming MapReduce job executed a word count on the input data.
- The output correctly computed word frequencies across all input lines.
- Parallel execution was observed across both map and reduce tasks.
- The larger dataset test demonstrated the scalability benefit of distributed processing.

---

**Discussion and Conclusion**

This lab implemented the MapReduce programming model using Amazon EMR. The mapper decomposed input text into key-value pairs in the form `(word, 1)`, and the reducer aggregated those pairs to produce a final count for each unique word. Hadoop handled data distribution, fault tolerance, and the shuffle and sort phase transparently.

MapReduce achieves parallel efficiency primarily through data locality: mappers run on the nodes where the data already resides, which reduces network transfer and speeds up execution. Unlike thread programming, which is constrained to a single machine's cores, or task programming, which is limited to a single machine's resources, MapReduce scales horizontally across hundreds of nodes. This makes it well-suited for enterprise batch workloads such as log analysis, ETL pipelines, and large-scale data aggregation. Amazon EMR reduces the operational burden of running these workloads by automating cluster management entirely.

---
