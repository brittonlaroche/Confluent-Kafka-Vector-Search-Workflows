# Step 3.  Workflows

Workflows are often a second thought on most Generative AI applications.  Its critical to capture all the different possibilities and places that a GenAI app will need to get data.  Not all of it comes from the Vector Database, and often there are better, easier more effective and effcient ways to get things done. 

### General Pattern
![Workflows Genreral Architecture](/files/img/workflowsGeneralPattern2.png)  

Workflows are a critical part to solving a users question by getting the right answers from the right places.  Maybe we need to call the LLM maybe we don't, maybe we need to do a vector search maybe we do not.  Maybe we can save on cost if we route the questions more effectively or cache the results to common questions.  Focusing on this retail example lets consider the following questions.

"What time does the store on Stonebrook Pkwy open tomorrow?"   
"How many reward points do I have?"   
"Show me your cheapest mens formal dress shirts."   
"I need to buy some formal dress shoes now, where can I go?"   
   
In some cases we need to get information from the product vector database.  In some cases we need to get user profile data. In some cases we need to get information on the physical location and hours of operation of retail store.  Some times we need to combine all three types of data into a vector search.
   
### Implementation Detail
![Workflows Genreral Architecture](/files/img/workflowsImplementation.png)  



## 4 Steps to Building a GenAI Application
There are 4 steps to building a GenAI Application and I have included a github for each step.    
The githubs (some still work in progress) are indexed here:   
Step 1. Data Augmenatation: [Confluent-Kafka-Vector-Encoding](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Encoding)   
Step 2. Inference: [Confluent-Kafka-Vector-Search-Prompt-Inference](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Prompt-Inference)   
Step 3. Workflows: [Confluent-Kafka-Vector-Search-Workflows](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Workflows)   
Step 4. Post Processing: [Confluent-Kafka-Vector-Search-Post-Processing](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Post-Processing)   
   
