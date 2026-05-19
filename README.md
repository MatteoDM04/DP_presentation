# Technical Challenges Presentation Draft


## Slide 1: Technical challenges

**Spoken Text:**
"Now I'm going to walk you through the technical challenges that we faced. There are 3 big components to our project. I will first talk about the solution storage where operators can document a solution to a machine issue that they fixed using voice or text and then the industrial chatbot where operators can ask all kinds of different questions about the machines to help them diagnose issues faster. Afterwards, Senne will walk you through the challenges of our preventive maintenance component. 

---

## Slide 2: Challenge 1 - Solution Storage & Information Extraction

**Spoken Text:**
"Let's start with Solution Storage. Our first challenge was converting raw voice input into text. While we used the OpenAI Whisper platform for basic speech-to-text, this is too general and not good enough for a noisy industrial setting where domain jargon would often get transcribed wrong. We solved this by first loading a custom Jargon Prompt that acts as a cheat sheet for Whisper's transcription, helping it to recognize specific industrial terminology like machine names and components. 
However, getting the correct raw text was only a small part of the challenges. Now we needed to extract the necessary information out of this text. To do this, the component evolved into three different stages.

Initially we tried a python library called Spacy, specialized for industrial entity extraction. It identified machines and components well, but completely failed to capture complex operator actions. To improve this we switched over to LLM information extraction, which worked quite well in the beginning. But after testing we noticed that this LLM struggled to map the unstructured text into 9 distinct components. Crucially, it couldn't separate the actual fix from passive diagnostic steps like checking a sensor that was fine. We solved this by developing a 'Two-Pass Extraction' architecture where we split the information extraction up into 2 different steps. 
The first pass queries a custom vector embedding database of all machine manuals to construct a true chronological timeline of events that filters out diagnostic noise. Then in the second pass everything gets structured into a formal JSON format that clearly captures everything that was done on which components. This also gets paired with 3D models and images of machine parts that can get added to the solution for extra information.

Lastly, because we couldn't just dump AI-extracted solutions directly into the database since this would become very unstructured very fast. We also built a human in the loop review workflow, where solutions are temporarily held in a pending state until they get verified by an supervisor."

---

## Slide 3: Challenge 2 - Dialect & Semantic Similarity

**Spoken Text:**
A second big challenge in this component was handling the different dialects and wordings which is very important to prevent the system from getting full with same solutions being explained in different wordings. If one operator explains a fix as 'Swapped the container of the buggy' and another says 'Replaced buggy container', then keyword matching would fail. We solved this by building a Hybrid system where we use a deep learning model called Sentence Transformers that converts the raw text into vector embeddings and calculates the Cosine Similarity based on concept and meaning. This naturally maps synonyms and dialects close together. However, the problem with this is that sentence structures in solutions can become so different that the comparison score can drop a lot. This is why we run a parallel check using our Two-Pass Extraction architecture. Here, different words like 'swap', 'replace', 'change' get mapped to the same action. The weakness of this second part is that new slang words that are not in the dictionary might fail to get extracted. But by combining it with the deep learning brain of Sentence Transformers and calculating both scores, we can get highly accurate similarity matching that understand the sentence context while simultaneously stripping away all unneccesary noise.

## Slide 4: Challenge 3 - RAG Pipelines and Agentic Tool Use


**Spoken Text:**
"Moving over to the Chatbot, the main technical difficulty here was that it isn't just a static conversational model. It acts as an intelligent agent. To ensure accuracy and to prevent AI hallucinations, we built a robust RAG pipeline. Instead of relying on the LLM's reasoning, we engineered it to actively retrieve verified solutions and machine manuals from our database. The true complexity here lies in the tool use. The chatbot dynamically decides when to execute a database search, when to look for previously stored solutions or when to activate a deeper "Thinking mode" for complex diagnostics. Getting the LLM to consistently use the right tools was a complex task and required careful prompt engineering to prevent the agent from hallucinating or getting stuck in loops. This paired with the live gathering of error logs and thus being able to filter out hundreds of unhelpful information logs made this a component that was invested a lot of time in to ensure the necessary accuracy.

---


**Spoken Text:**
"Even with RAG and tools, the AI sometimes fails to find a perfect match. Another big challenge was getting the chatbot to know when it *didn't* know the answer. We had to avoid two dangerous extremes of 'ghost confidence' and 'false refusals'.
Firstly, standard LLM's tend to carry over confidence from previous questions. This 'ghost confidence' can lead to the AI to confidently double down on wrong answers. We avoided this by decoupling the confidence scores from chat history and strictly basing them on the current query and the actual retrieved RAG data and tools used. 
Secondly, we had to prevent 'false refusals'. We had to differentiate: Is the operator asking an off-topic question, or are they asking a valid machine question that simply lacks database documentation?
We solved this by developing a multi-layered confidence algorithm. Instead of a flat refusal, it blocks off-topic queries, penalizes vague questions and guides the operator to provide more specific machine information to improve the next RAG search. 

"
---

QUALITY CONTROL:
To guarantee quality in our project, I personally put a lot of time into bug fixing and ensuring the non-functional requirements were met. These main requirements include mainly the Solution Capture Accuracy and the Filtering of different dialects, then the accuracy of all chatbot answers and system wide performance and responsiveness. But guaranteeing that these were all met was definitely more of a group effort than just the doing of just myself where everyone was responsible for guaranteeing quality in the components they are working on. We also put a mandatory merge approval system in place where every merge to our development branch had to be checked by someone else to ensure no quality gets lost by accidental changes. 
