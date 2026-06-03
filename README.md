# CST8917-Assignment1-ShanJiang

## README.md

### Header
* **Name**: Shan Jiang
* **Student Number**: 041179466
* **Course Code**: CST8917
* **Assignment Title**: Assignment 1 - Serverless Computing - Critical Analysis
* **Date**: June 2, 2026

---

### Part 1: Paper Summary
#### 1. Main Argument
The central thesis of the paper is about the major problems in first-generation serverless computing, which is also called Functions-as-a-Service or FaaS. The authors use the phrase "one step forward, two steps back" to describe the current state of this technology. The "one step forward" refers to the great features of auto-scaling and pay-as-you-go billing. Developers do not need to manage servers or worry about infrastructure setup anymore. They only pay for the exact compute resources they use when their code runs. This is a big step forward for cloud programming ease. However, the "two steps back" means that current FaaS designs hurt modern computing trends. These trends are data-centric computing and distributed systems innovation.
#### 2. Key Limitations Identified
The authors identify several key limitations in current FaaS platforms like AWS Lambda. 
- First, there are strict execution time constraints. Functions are killed automatically after 15 minutes. This makes it very hard to save or recover state across multiple function calls. 
- Second, there are huge network and communication limitations. Functions do not have direct network addresses. Because of this lack of addressability, two running functions cannot talk directly to each other. They must use slow intermediate cloud storage like S3 or DynamoDB to pass messages. Also, there is a bad network I/O bottleneck. When many functions run on the same virtual machine, they must share the network bandwidth, so the speed becomes much slower than a single standard SSD. 
- Third, these limitations create a "data shipping" anti-pattern. Good traditional data systems try to send code to where the data lives. But FaaS does the opposite. It always moves large data blocks over the slow network to the isolated, stateless code. This causes high latency and costs too much money. 
- Fourth, FaaS gives limited hardware access. It only allows standard CPU timeslices and small RAM options. Developers cannot use specialized hardware like GPUs or big main-memory databases. 

Finally, all these problems stop developers from building good distributed systems. Basic distributed protocols like leader election or consensus become extremely slow and expensive because every communication must go through cloud storage. 
#### 3. Proposed Future Directions
- First proposal is "Fluid Code and Data Placement". The cloud platform should automatically colocate code and data by shipping code to the data side.
- Second proposal is "Heterogeneous Hardware Support". Future systems should let high-level languages compile code onto specialized hardware like GPUs or custom chips. 
- Third proposal is "Long-Running, Addressable Virtual Agents". Cloud programs need virtual software agents that live for a long time with fixed network identities so they can talk to each other with fast, direct network performance.

---

### Part 2: Azure Durable Functions Deep Dive

#### Topic 1: Orchestrator Functions and State Management
Orchestrator functions are a special type of function in Azure Durable Functions. They let you define complex workflows using normal procedural code. The best capability is that the Durable Functions runtime automatically manages state, checkpoints, and restarts for you. Our course slides show that regular serverless functions have a big challenge with stateless execution. The paper criticized first-generation FaaS because functions lose state and cannot keep data across different calls. Orchestrator functions directly solve this criticism. They automatically save the state of the workflow behind the scenes. Developers do not need to manually write state tracking code into a separate slow database after every small step, which reduces the paper's complaints about managing state.

#### Topic 2: Activity Functions and Unit of Work
Activity functions are the basic units of work in an Azure Durable Functions application. They are the actual functions that do the real business logic or data tasks, and they are called by orchestrator functions. Our course slides mention that serverless functions are usually designed to be small, single-purpose, and stateless. The paper criticized old FaaS platforms because short-lived functions lack coordination and cannot work together efficiently on big data jobs. Activity functions help because they break a large, complex job into small, safe tasks. The orchestrator tracks their execution. If an activity function fails, the orchestrator knows exactly where it stopped and can retry it. However, activity functions themselves are still stateless, so they still face the paper's criticism about I/O bottlenecks when downloading large files.

#### Topic 3: Client Functions and Starting Workflows
Client functions are regular Azure Functions that start an instance of an orchestrator function. For example, you can use an HTTP-triggered function as a client function. The course slides state that a function chain can be initiated by an external event or a manual trigger. The paper strongly criticized serverless platforms because a client cannot address a specific running function instance directly, meaning there is no connection stickiness. Client functions partially help fix this issue. When a client function starts an orchestrator, it gets back a unique instance ID. The client can use this ID to check the real-time status or send new events to that specific running workflow instance later. This gives developers a way to address a running process.

#### Topic 4: Function Chaining Pattern
The function chaining pattern means executing a sequence of functions in a specific order. Each function runs its task, finishes, and then triggers the next function while passing its output data as the next input. The course slides explain that chaining lets you create more sophisticated and capable applications from small functions. The paper criticized function chaining in first-generation FaaS because it has very high latency and bad performance. This happens because functions must pass data through slow intermediate storage systems. Azure Durable Functions makes this pattern much easier to write as code. But beneath the surface, it still uses storage queues and tables to pass messages, so the paper's criticism about slow communication media remains true.

#### Topic 5: Error Handling and Built-in Retries
Managing the sequence and handling errors or retries in standard serverless functions adds a lot of code complexity. Azure Durable Functions provides built-in orchestration tools to handle errors and do automatic retries using simple code. The paper noted that in event-driven systems, if something fails, dealing with global state consistency across ephemeral functions is very hard and requires extra complex protocols. Durable Functions addresses this challenge. It lets developers write standard try-catch blocks or call retry policies directly inside the orchestrator code. This stops the developer from having to manually build complex error-handling or retry logic into every single lambda function, making stateful workflows more reliable when cloud hardware fails.

---

### Part 3: Critical Evaluation

#### 1.Limitations that remain unresolved
Although Azure Durable Functions gives developers an easier way to write stateful workflows, it does not solve all the deep hardware and network problems mentioned by the authors. I will highlight two criticisms from the paper that are still unresolved or only partially fixed by this technology.

The first unresolved limitation is the "Data Shipping" anti-pattern and the network I/O bottleneck. The paper states that moving large data over a network to stateless code is a bad design choice that causes high latency and high cost. Azure Durable Functions lets you write the control flow easily, but the physical location of the data and code remains separate. Activity functions still run on separate virtual machines away from the main data storage. Every time an activity function needs data, it must fetch it over the network from a service like Azure Blob Storage or a database. Also, the Durable Functions runtime itself depends heavily on an underlying storage provider to save checkpoints and queue messages. This means every step in the chain creates many read and write operations to cloud storage, which keeps the network I/O bottleneck alive.

The second unresolved limitation is the lack of specialized hardware support. The paper explains that modern data-intensive applications, like machine learning model training, need access to specialized hardware like GPUs or massive RAM. Standard serverless functions usually only allow standard CPU slices and small memory amounts. Azure Durable Functions does not change this hardware limitation for its basic consumption plans. If you run your activity functions on a cheap pay-as-you-go plan, you still cannot use GPUs or keep huge main-memory databases active. Therefore, you cannot run heavy deep learning training inside this framework efficiently, just like the slow TensorFlow example in the paper.

#### 2.Your Verdict
My verdict is that Azure Durable Functions represents a clever software workaround rather than the true architectural progress the authors envisioned for serverless computing. 

The authors wanted a total change in cloud infrastructure, like fluid code placement where the cloud automatically pushes code to where the data lives. They also wanted true long-running virtual agents with fast, direct point-to-point network communication. Azure Durable Functions does not change the underlying physical cloud architecture. It takes normal stateless functions and connects them using hidden storage queues and tables. It hides the complexity of state management from the developer's eyes, which is very helpful for building regular business applications. But underneath, the system is still shipping data across racks, facing cold starts, and paying storage latency costs. It is a very good orchestration tool, but it does not fix the deep physical limitations of cloud hardware described in the paper.

---

### References

1. Hellerstein, J. M., Faleiro, J., Gonzalez, J. E., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). *Serverless Computing: One Step Forward, Two Steps Back*. CIDR 2019. [https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf](https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf)
2. Microsoft Corporation. (2026). *Durable Functions Overview: Stateful Serverless Workflows - Microsoft Learn*. [https://learn.microsoft.com/en-us/azure/azure-functions/durable-functions/durable-functions-overview](https://learn.microsoft.com/en-us/azure/azure-functions/durable-functions/durable-functions-overview)
3. Microsoft Corporation. (2026). *Azure Functions Overview | Microsoft Learn*. [https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview)
4. CST8917 Course Slides. (2026). *Week 3 - Chaining Azure Functions*. Algonquin College Course Material.

---

### AI Disclosure Statement
I used AI tools (Gemini) to complete this assignment：For Part 2, I used the AI to connect the Azure Durable Functions features from our Week 3 course slides to the specific criticisms in the paper. Organize the grammatically incorrect sentences in the assignment and convert them to MD format.