# Technical Challenges Presentation Draft

## Slide 1: Introduction to Our Technical Journey
**Visual Suggestion:** A high-level schematic showing the two main pillars of the system: The **Solution Storage Pipeline** on the left (with icons for voice/text input flowing into a database) and the **Industrial Chatbot** on the right (with an icon of an operator interacting with the AI).

**Spoken Text:**
"Hi everyone. Today I'm going to walk you through the technical engine room of our project. While the user interface makes the process look seamless, there’s a lot of complexity under the hood. Specifically, I want to talk about the technical challenges we faced when building the two core components: the Solution Storage system and the Conversational Chatbot, and how we engineered our way around them."

---

## Slide 2: Challenge 1 - Solution Storage & Information Extraction
**Visual Suggestion:** A step-by-step pipeline diagram.
1. Raw Input (Voice/Text) -> 
2. "Two-Pass Extraction Engine" (Highlight this as the complex part) -> 
3. "Admin Review Queue" -> 
4. Knowledge Base.

**Spoken Text:**
"Let's start with Solution Storage. The platform provides basic data storage, but our challenge was getting *clean, actionable* data into it. Operators might submit solutions via voice or messy text. 

The biggest difficulty was extracting the exact entities and steps reliably without losing context. We initially faced issues with the AI failing to parse data correctly, leading to serialization crashes and lost information. To solve this, we implemented a 'Two-Pass Extraction' logic. The first pass captures the raw transcribed text to preserve the original intent, and the second pass structurizes it into a formal JSON format. 

Another challenge was data integrity. We couldn't just dump AI-extracted solutions directly into the database. We mitigated this by building a 'Human-in-the-Loop' review workflow, where solutions are temporarily held in a pending state until an admin verifies them in the UI."

---

## Slide 3: Challenge 2 - Chatbot Reasoning and Confidence
**Visual Suggestion:** A flowchart showing the Chatbot's decision-making process.
User Query -> Log Retrieval -> Confidence Scorer -> Decision Split (High Confidence vs. Low Confidence Fallback).

**Spoken Text:**
"Moving over to the Chatbot, the main difficulty wasn't getting the AI to talk—it was getting it to know when it *didn't* know the answer. 

Early on, our chatbot would sometimes confidently give wrong answers, or conversely, refuse to answer valid technical questions, claiming they were 'out of scope' just because it had low confidence. 

We solved this by engineering a custom Confidence Scoring system. Instead of generic refusals, we built a domain-specific fallback. If the chatbot's confidence score drops below 25%, it doesn't just give up; it systematically asks the operator for more specific machine context. We also had to heavily optimize our log retrieval mechanism so the chatbot prioritized recent 'Errors' over generic 'Warnings', ensuring it was always reasoning over the most critical, up-to-date data."

---

## Slide 4: Challenge 3 - Conversational Context
**Visual Suggestion:** A visual representation of "Memory". A stack of chat bubbles where older ones fade out, but key "Technical Procedures" are highlighted and kept active.

**Spoken Text:**
"Another major hurdle with the chatbot was managing conversational memory. In an industrial setting, operators ask follow-up questions, like 'Can you repeat step 2?' 

The challenge is that Large Language Models can easily lose the technical thread if the conversation drifts. We mitigated this by refining how we handle the conversation history. We decoupled the confidence metrics from the global chat history, meaning the AI evaluates each new question on its own merits rather than being biased by the previous chat. This ensured the chatbot could fluidly repeat or summarize technical procedures without hallucinating or forgetting the machine context."

---

## Slide 5: Top 5 Lessons Learned
**Visual Suggestion:** A clean, bold numbered list counting down from 5 to 1.

**Spoken Text:**
"To wrap up, here is our team's Top 5 technical lessons learned from building these components:

**5. Real-World Data is Messy:** We learned quickly that text encoding issues, formatting bugs, and raw voice transcriptions will break your pipeline if you don't build robust, multi-pass data handlers.

**4. Context is Everything, but Hard to Manage:** Keeping a chatbot on track requires strict programmatic guardrails, not just basic prompt engineering. You have to explicitly code what it remembers.

**3. Prioritize Signal Over Noise:** When feeding logs to an AI, sending everything causes it to timeout or get confused. Filtering for 'Errors' over 'Warnings' was crucial for performance.

**2. Human-in-the-Loop is Mandatory:** No matter how good the AI extraction is, an Admin Review step is essential to prevent bad data from polluting your knowledge base permanently.

**1. Graceful Degradation over Hard Failures:** The biggest lesson was handling uncertainty. A low-confidence AI shouldn't just say 'I don't know'. It should be engineered to ask the right follow-up questions to get back on track."
