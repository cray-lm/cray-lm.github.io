# Using Large Language Models for Linux Code Repair

## Introduction

Recent advancements in Large Language Models (LLMs) have unlocked new opportunities for automated code repair, presenting innovative ways to simplify the maintenance of complex codebases. In this post, we delve into a practical case study showcasing how LLMs can be leveraged to identify and resolve issues in Linux kernel code.

## Why Code Repair Matters in Linux Development?
Linux development is uniquely challenging compared to other software projects. Below are some of the key reasons:

### 1. The Scale and Complexity of the Linux Codebase
The Linux kernel stands as one of the largest open-source projects globally, comprising millions of lines of code maintained by thousands of contributors. This immense scale brings substantial challenges in identifying and fixing bugs, as even minor issues can cascade into significant disruptions across modules and subsystems.

### 2. The Need for High Reliability and Security
Linux serves as the backbone for critical infrastructure, from servers and supercomputers to embedded devices and smartphones. Any bug or security vulnerability can lead to catastrophic outcomes, such as system failures, data breaches, or service interruptions. Achieving and maintaining high reliability and security is non-negotiable, making accurate and timely code repair a core requirement.

### 3. The Dynamics of a Community-Driven Open-Source Ecosystem
The open-source nature of Linux fosters a collaborative environment, welcoming contributions from developers worldwide. While this drives innovation, it also introduces challenges:

- Contributions vary widely in quality and expertise.
- Peer review processes may occasionally overlook subtle bugs or edge cases.
- Maintaining consistency and adherence to coding standards across diverse submissions is vital.


## How can of Code Repair alleviate these issues?
Automated code repair addresses these pain points by:

- Detecting bugs early in the development cycle to reduce downstream impact.
- Enhancing code quality and maintainability across the codebase.
- Relieving the workload on human reviewers by automating parts of the debugging process.
- Strengthening security through rapid patching of vulnerabilities before exploitation becomes feasible.
By streamlining the code repair process, the Linux community can uphold its legacy of reliability and innovation while scaling to meet future challenges.

## Static Code Analysis
Static code analyzers assess code without executing it, empowering teams to deliver better software faster. Here's how they add value:
### Key Benefits

#### 1. Catch Bugs Early
* Detects issues like null pointers and memory leaks during development
* Saves time and money by preventing runtime failures

#### 2. Maintain Code Quality
* Ensures consistent coding standards across large teams
* Identifies complex or duplicated code for cleanup

#### 3. Strengthen Security
* Identifies vulnerabilities before deployment
* Flags risky dependencies and unsafe coding patterns

### How It Works
Static code analysis combines several techniques to detect issues:

- AST Analysis: Parsing code into tree structures
- Control Flow Analysis: Mapping execution paths
- Data Flow Analysis: Tracking how data moves through the code
- Pattern Matching: Spotting problematic patterns

These techniques are augmented with algorithms like symbolic execution, taint analysis, type inference, and complexity calculations to detect potential bugs and vulnerabilities.

## TODO: Should we have an image instead?
Static code analysis -> Bug reports (Dataset) -> train(LLMs) -> Code Repair AI


## Enhancing Bug Fixing with Static Code Analysis
In traditional software enterprises, static code analysis tools scan large and complex codebases to find bugs. These tools not only identify potential issues but also recommend fixes based on best practices. However, reviewing and addressing these bug reports manually can be time-consuming and mundane, detracting from developers' focus on innovative tasks like feature development.

Automating the bug-fixing process using Large Language Models (LLMs) offers an exciting and efficient solution to this problem.

### Quality Inputs for Better Outcomes
To ensure robust results, we utilize workflows that generate high-quality bug reports, curating them into a rich dataset for model training.

## Leveraging Large Language Models for Code Repair
LLMs have revolutionized software development, excelling in tasks such as generating new code. However, their performance with complex legacy codebases has limitations. To bridge this gap, we adopt the Cray-LM unified framework to train state-of-the-art pre-trained models (e.g., LLaMA) using our curated bug report dataset.

Integrating LLMs into the code repair pipeline boosts productivity, minimizes errors, and enhances security by automating the resolution of vulnerabilities.

## Case Study: Repairing Buffer Overflow errors

### What are buffer overflow errors?
Buffer overflow occurs when a program writes more data than a memory buffer can hold, overwriting adjacent memory locations. This vulnerability is common in languages like C, which lack automatic bounds checking. Buffer overflows can be classified into three main types:

**1. Stack-Based Buffer Overflows**
Occur when data exceeds the bounds of a buffer on the stack. Can overwrite local variables and function return addresses, creating security risks.

**2. Heap-Based Buffer Overflows**
Occur in dynamically allocated memory on the heap. May corrupt heap management structures, which, though harder to exploit, can still have severe consequences.

**3. Integer Overflow Leading to Buffer Overflow**
Result from arithmetic operations that "wrap around," leading to incorrect buffer sizes.
Can cause data to be written beyond the intended memory bounds.

### Security Risks
#### 1. Code Execution
Attackers can inject and execute malicious code by overwriting memory. Manipulating return addresses enables them to control program execution, especially critical in kernel space where code runs with the highest privileges.

#### 2. System Crashes
Corrupted memory structures can cause system crashes or panics, particularly in kernel space. This often leads to denial-of-service (DoS) conditions.

#### 3. System Stability
Buffer overflows can cause unpredictable program behavior, threatening the stability of both individual applications and entire systems, especially in critical environments.

### Setup
#### Bug report dataset
Bug reports generated by static code analysis tools typically include details such as the source file name, bug type, line number, description of the issue, and a suggested fix.

Here’s an example:

```
CID: 1845632
Type: Buffer overflow
Category: BUFFER_OVERFLOW
Classification: Bad use of string function
Severity: High
Certainty: Absolute
Status: New
Function: cxl_mem_create_range_info
File: drivers/cxl/core/mem.c
Line: 723

Issue:
Unbounded sprintf can cause buffer overflow. The code uses sprintf() without size limits to write into buffer 'range_id', which could lead to buffer overflow if the formatted string exceeds the buffer size.

Description:
The function cxl_mem_create_range_info() uses sprintf() to format a memory range identifier into a string buffer 'range_id' without checking if the resulting string will fit in the destination buffer. This could lead to a buffer overflow if the range address and size generate a string longer than the size of range_id.

Use snprintf instead
```

#### LLM training
For each bug report, a manual fix was created to address the issue. These fixes were formatted as git diffs, creating pairs of bug reports and corresponding code changes to serve as a training dataset for a Large Language Model (LLM).

Example diff for the bug report above:

```
diff --git a/drivers/cxl/core/mem.c b/drivers/cxl/core/mem.c
index 9af4721c5e32..b7d45890c453 100644
--- a/drivers/cxl/core/mem.c
+++ b/drivers/cxl/core/mem.c
@@ -720,7 +720,7 @@ static int cxl_mem_create_range_info(struct cxl_dev_state *cxlds)
       struct cxl_memdev *cxlmd = to_cxl_memdev(cxlds->dev);
       char range_id[32];

-       sprintf(range_id, "0x%llx-%llx", range->start, range->size);
+       snprintf(range_id, sizeof(range_id), "0x%llx-%llx", range->start, range->size);
       cxlds->range_id = kstrdup(range_id, GFP_KERNEL);
       if (!cxlds->range_id)
               return -ENOMEM;
```

Using just a dozen highly curated samples, we fine-tuned a pre-trained model on this task. When evaluated on a held-out test set, the diffs generated by the trained model demonstrated strong quality and alignment with expert-created fixes.


### Limitations and Challenges
While the results are promising, there are notable limitations and challenges:

#### 1. Diverse Bug Representation
The effectiveness of the model heavily depends on the diversity of bug types in the training set. For example, if a bug type is underrepresented, the model may struggle to generalize fixes for similar issues.

#### 2. Context Constraints
To provide accurate fixes, the necessary context for the bug must be small and contained within a single source file. Handling bugs that span multiple files or require a broader understanding of the codebase is beyond the current model's capabilities.

#### 3. Curating Training Data
Creating the bug report and fix dataset requires significant manual effort by experts deeply familiar with the codebase. This process is time-intensive and limits scalability.

### Conclusion
This approach demonstrates the potential of using a small, highly curated dataset to fine-tune LLMs for automated code repair tasks. The trained model delivers high-quality fixes on a held-out test set, showcasing the value of combining expert domain knowledge with machine learning. However, further efforts are needed to address the limitations in bug diversity, context handling, and dataset scalability.

### Future Work
To advance this work:

#### 1. Expand the Dataset
Collaborate with domain experts to create a more comprehensive and diverse training dataset.
#### 2. Address Multi-File Context
Investigate techniques to incorporate broader code context for bugs that span multiple files.
#### 3. Automate Dataset Creation
Explore semi-automated methods to identify bugs and generate preliminary fixes, reducing manual effort.

Together, these efforts can improve the applicability and robustness of LLM-based bug repair systems, paving the way for their broader adoption in software development workflows.

---

*Have thoughts or experiences with using LLMs for code repair? Send us a message to get involved!*
