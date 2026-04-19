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
  - The intermediate key-value pairs are grouped by key and forwarded to the appropriate reducers.

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
  1. Go to **AWS Console** -> Search **S3** -> Click **Create bucket**
  2. Set **Bucket name**: `emr-lab-<your-name>` (replace `<your-id>` with your name)
  3. Bucket type -> **General purpose**
  4. **ACLs disabled**
  5. Leave all other settings as default -> Click **Create bucket**
  6. Create a folder named `logs` inside `input` folder. 

**1b. Create the Input File Locally**

On your local machine, create `input.txt`:
>Linux/Unix: vim input.txt

>Windows: notepad input.txt
```text name=input.txt
Cloud computing is the future of computing
Cloud services include IaaS PaaS and SaaS
Virtualization enables cloud computing
Cloud computing provides scalability and elasticity
```

**1c. Upload to S3**
  1. Open your bucket -> Click **Create folder** -> Name it `input` -> Click **Create folder**
  2. Click into the `input/` folder -> Click **Upload**
  3. Click **Add files** -> Select your `input.txt` file
  4. Click **Upload**

>*Screenshot checkpoint: S3 bucket showing `input/input.txt`*

>Sample:

<img width="1459" height="340" alt="Screenshot 2026-04-19 at 11 37 10 AM" src="https://github.com/user-attachments/assets/3db53285-daa6-4d92-9220-3575e361ce07" />


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
| **Core instance count** | `2` |
| **Cluster configuration** | **Uniform instance groups** |
| **Cluster scaling** | **Set cluster size manually** |
| **Networking — VPC** | Leave default selected |
| **Networking — Subnet** | Leave default selected  |
| **Security groups** | Leave as **EMR-managed** for both Primary and Core nodes |
| **Cluster termination** | Automatically terminate cluster after idle time |
| **Idle time** | `01:00:00` (1 hour) |
| **Termination protection** | **Off** |
| **EC2 Key pair** | Select your existing key pair from the dropdown |
| **Cluster logs-S3 location** | `s3://emr-lab-<your-id>/logs/` (`Copy S3 uri` of logs folder and paste) |
| **Service role** | `EMR_DefaultRole` |
| **EC2 instance profile** | `EMR_EC2_DefaultRole` |
| **EBS root volume size** | `15 GiB` (default) |
| **Operating system** | Amazon Linux (default, latest updates applied) |

5. Click **Create cluster**

**Step 3: Wait for the Cluster to Become Ready**

  1. You'll be redirected to the cluster detail page
  2. Watch the **Status** badge cycle through: - 🔵 `Starting` → 🟡 `Bootstrapping` → 🟢 **`Waiting`**
  3. This typically takes **8–15 minutes**

>*Screenshot checkpoint: Cluster showing Waiting status [Mandatory]*

>Sample:

<img width="1455" height="714" alt="Screenshot 2026-04-19 at 12 30 56 PM" src="https://github.com/user-attachments/assets/18cf0e5a-b0f9-461a-9645-483b1c40a0c8" />


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

  1. Go to your S3 bucket → Click **Create folder** → Name it `scripts` → Click **Create folder**
  2. Click into `scripts/` folder → Click **Upload**
  3. Click **Add files** → Select **both** `mapper.py` and `reducer.py`
  4. Click **Upload**

>*Screenshot checkpoint: S3 `scripts/` folder showing both files*

>Sample:

<img width="1461" height="470" alt="Screenshot 2026-04-19 at 12 36 12 PM" src="https://github.com/user-attachments/assets/a1cc467f-b466-4d94-ab8e-1167e8c5928a" />


**Step 5: Add a Streaming Step to the Cluster**

1. Go to **EMR Console** → Click your `MapReduce-Lab-Cluster`
2. Click the **Steps** tab → Click **Add step**
   
> Sample:

<img width="1457" height="427" alt="Screenshot 2026-04-19 at 12 37 39 PM" src="https://github.com/user-attachments/assets/2a5760a0-5c9a-4116-8d67-5b87a5a47348" />


4. In the dialog, configure:

| Field | Value |
|---|---|
| **Step type** | `Streaming program` |
| **Name** | `WordCount-Job` |
| **Mapper** | `s3://emr-lab-<your-id>/scripts/mapper.py` |
| **Reducer** | `s3://emr-lab-<your-id>/scripts/reducer.py` |
| **Input location** | `s3://emr-lab-<your-id>/input/` |
| **Output location** | `s3://emr-lab-<your-id>/output/wordcount/` |
| **Action on failure** | `Continue` |

> The **output path must NOT already exist** in S3. Hadoop will create it automatically.

5. Click **Add**



**Step 6: Monitor Job Execution**

  1. Stay on the **Steps** tab of your cluster
  2. Watch the `WordCount-Job` step move through statuses:

```
Pending -> Running -> Completed
```

3. Click on the step name to view:
   - **Log files** (stdout, stderr, controller)
   - Direct links to YARN/Hadoop logs

> 💡 **Tip:** If the step shows **Failed**, click the step → open **stderr** log to diagnose the error.

>*Screenshot checkpoint: Step showing **Completed** status in green*



**Step 7: View the Output in S3**

1. Go to **S3** → Navigate to:
   ```
   s3://emr-lab-<your-id>/output/wordcount/
   ```
2. You'll see files like `part-00000`, `part-00001`, and `_SUCCESS`
3. Click **part-00000** → Click **Download**
4. Open the file — it should look like:

```text name=part-00000
and             2
cloud           4
computing       4
elasticity      1
enables         1
future          1
iaas            1
include         1
is              1
of              1
paas            1
provides        1
saas            1
scalability     1
services        1
the             1
virtualization  1
```

>*Screenshot checkpoint: S3 output folder and/or the downloaded file contents*



**Step 8: Examine Hadoop Job Logs and Metrics**

**Option A: Via EMR Application UIs (Recommended)**

1. Go to your cluster in EMR Console
2. Click **Application user interfaces** tab
3. Click **ResourceManager** (you may need to enable SSH tunnel or use the **SSM** option if available)

> **For console access without SSH:** EMR 6.x+ supports persistent application UIs via the console directly under **Application user interfaces → On-cluster UIs**. Click **Enable** if prompted.

4. In the ResourceManager UI, click **Applications → FINISHED**
5. Click your job to view:
   - Number of **Map tasks**
   - Number of **Reduce tasks**
   - **Total execution time**
   - **Input/output bytes processed**

### Option B: Via Step Logs in S3

1. Go to `s3://emr-lab-<your-id>/logs/<cluster-id>/steps/<step-id>/`
2. Open `stdout` and `stderr` for job summary info

>*Screenshot checkpoint: Hadoop ResourceManager job statistics page*


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


**Step 10: Terminate the EMR Cluster**

> **Critical:** EMR clusters charge by the hour. Always terminate when done.

1. Go to **EMR Console** -> Select your `MapReduce-Lab-Cluster`
2. Click **Actions** (top right) -> Click **Terminate**
3. A confirmation dialog appears - click **Terminate** again
4. The cluster status will change to `Terminating` → `Terminated`

*Screenshot checkpoint: Termination confirmation dialog or cluster in `Terminated` state*

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
