# CST8917-Assignment1-ShanJiang

## README.md

### Header
* **Name**: Shan Jiang
* **Student Number**: 041179466
* **Course Code**: CST8917
* **Assignment Title**: Assignment 1 - Serverless Computing - Critical Analysis
* **Date**: June 10, 2026

---

### Part 1: Paper Summary
#### 1. Main Argument
The central thesis of the paper is about the major problems in first-generation serverless computing, which is also called Functions-as-a-Service or FaaS. The authors use the phrase "one step forward, two steps back" to describe the current state of this technology. The "one step forward" refers to the great features of auto-scaling and pay-as-you-go billing. Developers do not need to manage servers or worry about infrastructure setup anymore. They only pay for the exact compute resources they use when their code runs. This is a big step forward for cloud programming ease. However, the "two steps back" means that current FaaS designs hurt modern computing trends. These trends are data-centric computing and distributed systems innovation.
#### 2. Key Limitations Identified
The authors identify several key limitations in current FaaS platforms like AWS Lambda. 
 - First, there are strict execution time constraints. Functions are killed automatically after 15 minutes, making it very hard to save or recover state across multiple function calls. 
 - Second, there are huge network and communication limitations because functions do not have direct network addresses. Because of this lack of addressability, two running functions cannot talk directly to each other. They must use slow intermediate cloud storage like S3 or DynamoDB to pass messages. Also, there is a bad network I/O bottleneck. When many functions run on the same virtual machine, they must share the network bandwidth, so the speed becomes much slower than a single standard SSD.
 - Third, these limitations create a "data shipping" anti-pattern. Good traditional data systems try to send code to where the data lives. But FaaS does the opposite. It always moves large data blocks over the slow network to the isolated, stateless code, causing high latency and high costs.
 - Fourth, FaaS gives limited hardware access. It only allows standard CPU timeslices and small RAM options without access to specialized hardware like GPUs. 
 
 Finally, all these problems stop developers from building good distributed systems. Basic distributed protocols like leader election become extremely slow and expensive because every communication must go through cloud storage.
#### 3. Proposed Future Directions
 - First proposal is "Fluid Code and Data Placement" to automatically colocate code and data by shipping code to the data side.
 - Second proposal is "Heterogeneous Hardware Support" to let high-level languages compile code onto specialized hardware like GPUs or custom chips.
 - Third proposal is "Long-Running, Addressable Virtual Agents" that live for a long time with fixed network identities for fast, direct network performance.

---

### Part 2: Azure Durable Functions Deep Dive

#### Topic 1: Orchestration Model
The orchestration model connects client, orchestrator, and activity functions together to run a stateful workflow. A client function triggers and starts the process from external events, like an HTTP request. The orchestrator function defines the workflow logical steps using normal procedural code, and it orchestrates the process by calling activity functions to perform the actual business tasks. This differs completely from basic FaaS. In basic FaaS, functions are isolated and independent, and developers must manually pass data using external databases. The orchestration model manages these components as a single workflow automatically. This addresses the paper's criticism that old FaaS platforms lack a structured framework to coordinate single-purpose, short-lived functions.

#### Topic 2: State Management
Durable Functions manages state automatically using event sourcing, checkpointing, and replay mechanisms. When the orchestrator calls an activity function, it logs the event and goes to sleep. Its execution history is saved automatically in an underlying Azure Storage table. When the activity finishes, the orchestrator wakes up, restarts, and replays the saved history to restore its state without running completed tasks again. Our course slides note that regular serverless functions face a big challenge with stateless execution. The paper criticized old FaaS because saving state requires writing data to slow databases manually. This automatic event sourcing capability addresses that criticism by hiding the state management complexity from the developer.

#### Topic 3: Execution Timeouts
Regular Azure Functions have strict execution timeouts, such as 5 to 10 minutes on a standard Consumption plan. Orchestrator functions successfully bypass these timeout limits because they do not run continuously. When an orchestrator awaits a long task, it unloads from memory, stops executing, and saves its state, allowing the overall workflow to safely run for days or months without timing out. However, standard timeout limits still apply to activity functions because they perform the actual physical work. If an activity function runs longer than the plan limit, it will still time out. This addresses the paper's criticism about the 15-minute limited lifetime of FaaS by allowing long-running control logic.

#### Topic 4: Communication Between Functions
Orchestrator and activity functions communicate using internal storage messages. When an orchestrator function calls an activity function, the framework automatically places a message into an underlying Azure Storage queue. The activity function pulls the message, completes the work, and passes the output data back through another queue. This design does not solve the paper’s criticism about slow storage intermediaries. The paper criticized FaaS because functions cannot communicate via fast, direct point-to-point network messaging. Durable Functions simplifies writing the code, but beneath the surface, it still relies heavily on slow storage queues and tables to pass data, meaning storage latency remains true.

#### Topic 5: Parallel Execution (Fan-out/Fan-in)
The fan-out/fan-in pattern runs multiple activity functions in parallel and then aggregates the results. Fan-out happens when the orchestrator triggers several tasks at the same time, and fan-in occurs when it waits for all those parallel tasks to finish using coding tools like Task.WhenAll. This pattern directly addresses the paper's concern that serverless platforms stymie distributed computing. The paper noted that executing uncoordinated parallel tasks is very difficult because functions cannot synchronize. Fan-out/fan-in solves this problem by letting developers write coordinated parallel applications easily using simple procedural code, and the runtime automatically manages the complex distributed task scheduling.

---

### Part 3: Critical Evaluation

#### 1. Limitations that remain unresolved
Although Azure Durable Functions gives developers an easier way to write stateful workflows, it does not solve all the deep hardware and network problems mentioned by the authors. I will highlight two criticisms from the paper that are still unresolved or only partially fixed by this technology.

The first unresolved limitation is the "Data Shipping" anti-pattern and the network I/O bottleneck. The paper states that moving large data over a network to stateless code is a bad design choice that causes high latency and high cost. Azure Durable Functions lets you write the control flow easily, but the physical location of the data and code remains separate. Activity functions still run on separate virtual machines away from the main data storage, meaning they must fetch data over the network from a service like Azure Blob Storage every time. Also, the Durable Functions runtime itself depends heavily on an underlying storage provider to save checkpoints and queue messages, which keeps the network I/O bottleneck alive.

The second unresolved limitation is the lack of specialized hardware support. The paper explains that modern data-intensive applications, like machine learning model training, need access to specialized hardware like GPUs. Standard serverless functions usually only allow standard CPU slices and small memory amounts. Azure Durable Functions does not change this hardware limitation for its basic consumption plans. If you run your activity functions on a cheap pay-as-you-go plan, you still cannot use GPUs or keep huge main-memory databases active. Therefore, you cannot run heavy deep learning training inside this framework efficiently, just like the slow TensorFlow example in the paper.

#### 2. Your Verdict
My verdict is that Azure Durable Functions represents a clever software workaround rather than the true architectural progress the authors envisioned for serverless computing. 

The authors wanted a total change in cloud infrastructure, like fluid code placement where the cloud automatically pushes code to where the data lives, and true long-running virtual agents with fast, direct point-to-point network communication. Azure Durable Functions does not change the underlying physical cloud architecture. It takes normal stateless functions and connects them using hidden storage queues and tables. It hides the complexity of state management from the developer's eyes, which is very helpful for building regular business applications. But underneath, the system is still shipping data across racks, facing cold starts, and paying storage latency costs. It is a very good orchestration tool, but it does not fix the deep physical limitations of cloud hardware described in the paper.

---

### References

1. Hellerstein, J. M., Faleiro, J., Gonzalez, J. E., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). *Serverless Computing: One Step Forward, Two Steps Back*. CIDR 2019. [https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf](https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf)
2. Microsoft Corporation. (2026). *Durable Functions Overview: Stateful Serverless Workflows - Microsoft Learn*. [https://learn.microsoft.com/en-us/azure/azure-functions/durable-functions/durable-functions-overview](https://learn.microsoft.com/en-us/azure/azure-functions/durable-functions/durable-functions-overview)
3. Microsoft Corporation. (2026). *Azure Functions Overview | Microsoft Learn*. [https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview)
4. CST8917 Course Slides. (2026). *Week 3 - Chaining Azure Functions*. Algonquin College Course Material.

---

### AI Disclosure Statement
I used AI tools (Gemini) to help me to finish this assignment. Specifically, I used the AI to help me reorganize Part 2 to match the five required specific technical topics (Orchestration model, State management, Execution timeouts, Communication between functions, and Parallel execution). I also used the AI to adjust the grammar of my sentences to keep the writing clear, direct, and well-structured in Markdown format.