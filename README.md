# Technical Challenges Presentation Draft

## Slide 1: Technical challenges
**Visual Suggestion:** A high-level schematic showing the two main pillars of the system: The **Solution Storage Pipeline** on the left (with icons for voice/text input flowing into a database) and the **Industrial Chatbot** on the right (with an icon of an operator interacting with the AI).

**Spoken Text:**
"Now I'm going to walk you through the technical challenges of our project. While the user interface makes the process look seamless, there’s a lot of complexity under the hood. Specifically, I want to talk about the technical challenges we faced when building the two core components: the Solution Storage system and the Conversational Chatbot, and how we engineered our way around them. Afterwards Senne and Alejandro will go through the technical challenges of the preventive maintenance and the database." 

---

## Slide 2: Challenge 1 - Solution Storage & Information Extraction
**Visual Suggestion:** A step-by-step pipeline diagram.
1. Raw Input (Voice/Text) -> 
2. "Two-Pass Extraction Engine" (Highlight this as the complex part) -> 
3. "Admin Review Queue" -> 
4. Knowledge Base.

**Spoken Text:**
"Let's start with Solution Storage. Our first challenge was converting raw voice input into text. While we used the OpenAI Whisper platform for basic speech-to-text, this is too general and not good enough for a noisy industrial setting. Using the basic Whisper, domain jargon would often be transcribed wrong. We solved this by first loading a custom Jargon Prompt that acts as a cheat sheet for Whisper's transcription, helping it to recognize specific industrial terminology like machine names and components. 
However, getting the correct raw text was only a small part of the challenges. Now we needed to extract the necessary information out of this text. To do this, the component evolved into three different stages.

Initially we tried a python library called Spacy, specialized for industrial entity extraction. It identified machines and components well, but completely failed to capture complex operator actions. To improve this we switched over to LLM information extraction, which worked quite well in the beginning. But after testing we noticed that this LLM struggled to map the unstructured text into 9 distinct components. Crucially, it couldn't separate the actual fix from passive diagnostic steps like checking a sensor that was fine. We solved this by developing a 'Two-Pass Extraction' architecture where we split the information extraction up into 2 different steps. 
The first pass queries a custom vector embedding database of all machine manuals to construct a true chronological timeline of events that filters out diagnostic noise. Then in the second pass everything gets structured into a formal JSON format that clearly captures everything that was done on which components. This also gets paired with 3D models and images of machine parts that can get added to the solution for extra information.

Lastly, because we couldn't just dump AI-extracted solutions directly into the database since this would become very unstructured very fast. We also built a human in the loop review workflow, where solutions are temporarily held in a pending state until they get verified by an supervisor."

---

## Slide 3: Challenge 2 - Dialect & Semantic Similarity
**Visual Suggestion:** A diagram showing two different operator quotes: Operator A: "Swapped the container buggy" and Operator B: "Replaced buggy container". Both point to Vector Embeddings that cluster together in Semantic Space, resulting in a "High Cosine Similarity Score (92%)".
**Spoken Text:**
A second big challenge in this component was handling the different dialects and wordings. If one operator explains a fix as 'Swapped the container of the buggy' and another says 'Replaced buggy container', then keyword matching would fail. We solved this by building a Hybrid system where we use a deep learning model called Sentence Transformers that converts the raw text into vector embeddings and calculates the Cosine Similarity based on concept and meaning. This naturally maps synonyms and dialects close together. However, the problem with this is that sentence structures in solutions can become so different that the comparison score can drop a lot. This is why we run a parallel check using our Two-Pass Extraction architecture. Here, different words like 'swap', 'replace', 'change' get mapped to the same action. The weakness of this second part is that new slang words that are not in the dictionary might fail to get extracted. But by combining it with the deep learning brain of Sentence Transformers and calculating both scores, we can get highly accurate similarity matching that understand the sentence context while simultaneously stripping away all unneccesary noise.

## Slide 4: Challenge 3 - RAG Pipelines and Agentic Tool Use
**Visual Suggestion:** A diagram showing the "Chatbot Agent" in the center, with arrows pointing to different "Tools" it can use: [Knowledge Base (RAG)], [System Error Logs], and [Thinking Mode].

**Spoken Text:**
"Moving over to the Chatbot, the main technical difficulty here was that it isn't just a static conversational model. It acts as an intelligent agent. To ensure accuracy and to prevent AI hallucinations, we built a robust RAG pipeline. Instead of relying on the LLM's reasoning, we engineered it to actively retrieve verified solutions and machine manuals from our database. The true complexity here lies in the tool use. The chatbot dynamically decides when to execute a database search, when to look for previously stored solutions or when to activate a deeper "Thinking mode" for complex diagnostics. Getting the LLM to consistently use the right tools was a complex task and required careful prompt engineering to prevent the agent from hallucinating or getting stuck in loops. This paired with the live gathering of error logs, filtering out hundreds of unhelpfull information logs makes this a very complex component.

---
## Slide 5: Challenge 4 - Chatbot Confidence and Fallbacks
**Visual Suggestion:** A flowchart showing the Confidence Scorer deciding between a "High Confidence Response" and a "Domain-Specific Fallback (Ask for Context)".

**Spoken Text:**
"Even with RAG and tools, the AI sometimes fails to find a perfect match. Another big challenge was getting the chatbot to know when it *didn't* know the answer. We had to avoid two dangerous extremes of 'ghost confidence' and 'false refusals'.
Firstly, standard LLM's tend to carry over confidence from previous questions. This 'ghost confidence' can lead to the AI to confidently double down on wrong answers. We avoided this by decoupling the confidence scores from chat history and strictly basing them on the current query and the actual retrieved RAG data and tools used. 
Secondly, we had to prevent 'false refusals'. We had to differentiate: Is the operator asking an off-topic question, or are they asking a valid machine question that simply lacks database documentation?
We solved this by developing a multi-layered confidence algorithm. Instead of a flat refusal, it blocks off-topic queries, penalizes vague questions and guides the operator to provide more specific machine context to refine the next RAG search. 

"
---

## Slide 6: Top 5 Lessons Learned
**Visual Suggestion:** A clean, bold numbered list counting down from 5 to 1.

**Spoken Text:**
"To wrap up, here is our team's Top 5 technical lessons learned from building these components:

Probably the most important lesson learned is that CONTEXT IS KING, but so difficult to manage. Keeping an AI agent with a lot of different tools on track requires strict guidelines to prevent hallucinations and getting stuck in loops.
Second lesson: Hybrid engineering beats pure AI. We learned that relying solely on deep learning or pattenr matching is a recipe for failure. By stacking multiple methods together, it is possible to create actual robust engines that none of these systems could have achieved on their own. 


# Presentation Visuals

Here are the upgraded, PowerPoint-optimized **widescreen (16:9)** Mermaid diagrams. These have been designed with a horizontal left-to-right flow (`flowchart LR`) or balanced split grids so they fit beautifully on slide templates without looking squished or requiring vertical scrolling.

---

## Slide 1: Technical Challenges Overview
A left-to-right split architecture showcasing the two parallel pillars of your system connected to a shared database.

```mermaid
flowchart LR
    subgraph Storage_Pipeline["Solution Storage Pipeline (Left Wing)"]
        A([Voice / Text Input]) --> B[Whisper STT Decoder]
        B --> C[Two-Pass Extraction Engine]
        C --> D[Admin Review Queue]
    end

    subgraph Knowledge_Base["Knowledge Base (Center DB)"]
        E[(Verified Solutions & manuals)]
    end

    subgraph Chat_Agent["Conversational Chatbot (Right Wing)"]
        F([Operator Query]) --> G[Agent Reasoning]
        G --> H{Confidence Scorer}
        H --> I[Technical Answer / Fallback]
    end

    D -->|Supervisor Approved| E
    E -.->|RAG Retrieval| G
    
    style Storage_Pipeline fill:#f4f4f4,stroke:#333,stroke-width:2px
    style Knowledge_Base fill:#e8f5e9,stroke:#2e7d32,stroke-width:3px
    style Chat_Agent fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
```

---

## Slide 2: Challenge 1 - Solution Storage & Extraction Pipeline
A sleek horizontal timeline detailing the three extraction stages, Jargon priming, and supervisor gating.

```mermaid
flowchart LR
    A([Operator Voice Input]) -->|Raw Audio| B[Whisper Speech-To-Text]
    B -->|Jargon Prompt Priming| C[Clean Transcribed Text]
    
    subgraph Two_Pass["Two-Pass Extraction Engine"]
        C --> D[Pass 1: Chronological timeline Ingestion]
        D -->|Diagnostic Noise Filtered| E[Pass 2: JSON Structure & Entity extraction]
    end
    
    E --> F[Pending Review Queue]
    F -->|Supervisor Approved| G[Main Database ]
    
    style Two_Pass fill:#fffacd,stroke:#8a6d3b,stroke-width:2px
    style G fill:#dff0d8,stroke:#3c763d,stroke-width:2px
```

---

## Slide 3: Challenge 2 - Dialect & Semantic Similarity
A widescreen flow diagram showing the parallel Sentence Transformer vector math check and the Two-Pass Action Dictionary check working in tandem.

```mermaid
flowchart TD
    subgraph Input_Jargon["Different Spoken Dialects"]
        A["Operator A: 'Swapped container of the buggy'"]
        B["Operator B: 'Replaced buggy container'"]
    end

    subgraph Dual_Check["Hybrid Semantic Matcher (Widescreen Parallel Pipeline)"]
        C{{"Deep Learning (all-MiniLM-L6-v2)"}} -->|1. Raw Text Cosine Similarity| E[sim_raw]
        D{{"Two-Pass Entity Matcher"}} -->|2. Structured Pattern Similarity| F[sim_pat]
    end

    A & B --> C
    A & B --> D
    
    E & F --> G[Max-Score Selector: max]
    G --> H[Apply Power Scaling: max ** 0.5]
    H -->|Result >= 80%| I[[Flag Duplicate / Map to Stored Solution]]

    style Dual_Check fill:#f5f5f5,stroke:#333,stroke-width:2px
    style I fill:#dff0d8,stroke:#3c763d,stroke-width:3px
```

---

## Slide 4: Challenge 3 - RAG & Agentic Tool Use
A hub-and-spoke layout showing the AI Agent dynamically choosing between tools to construct a clean context.

```mermaid
flowchart LR
    A([User Query]) --> B[Chatbot Agent Reasoning]
    
    subgraph Tool_Chest["Dynamic Tool Dispatcher"]
        B -->|Tool Choice 1| C[[RAG Search]]
        B -->|Tool Choice 2| D[[System Log Fetcher]]
        B -->|Tool Choice 3| E[[Deeper Thinking Mode]]
    end
    
    C --> F[(PDF Manuals & Solutions)]
    D --> G[(Live Machine logs)]
    E --> H[Multi-Step Diagnostic logic]
    
    F & G & H --> I[Context Synthesizer]
    I --> J[Proceed to Confidence Engine]
    
    style Tool_Chest fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
```

---

## Slide 5: Challenge 4 - Chatbot Confidence & Fallbacks
A horizontal flow diagram showing how you avoid false refusals and decouple global history to eliminate ghost confidence.

```mermaid
flowchart LR
    A[RAG Context + Current Query] -->|Decoupled from Global History| B{Confidence Scorer}
    
    B -->|Confidence >= 25%| C[High Confidence response]
    B -->|Confidence < 25%| D{Low Confidence Analyzer}
    
    C --> E[Output Technical Answer]
    
    D -->|Off-Topic Query| F[Block & Refuse]
    D -->|Valid but Vague Query| G[Domain-Specific Fallback: Ask for context]
    
    G -.->|User Refines Query| A
    
    style B fill:#f96,stroke:#333,stroke-width:3px
    style G fill:#fffacd,stroke:#8a6d3b,stroke-width:2px
    style F fill:#f99,stroke:#a94442,stroke-width:2px
```

---

## Slide 6: Top 5 Lessons Learned
A horizontal countdown sequence showing how your lessons flow together to build a robust system.

```mermaid
flowchart LR
    subgraph Lessons["Top 5 Lessons Learned (Countdown Sequence)"]
        L5["5. Messy Data (Jargon Priming)"] -->
        L4["4. Human review Guardrails"] -->
        L3["3. Graceful degradation Fallbacks"] -->
        L2["2. Hybrid beats Pure AI"] -->
        L1["1. Context is King (Decoupling)"]
    end
    
    style Lessons fill:#fafafa,stroke:#333,stroke-width:2px
    style L1 fill:#dff0d8,stroke:#3c763d,stroke-width:3px
```



