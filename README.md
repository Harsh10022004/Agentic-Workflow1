# Autonomous Academic Knowledge Extraction Workflow

**Loom Video Walkthrough:** `[Insert Your Loom Video Link Here]`

## 1. Problem Statement & Practical Usefulness
* **Target User:** University students, academic researchers, and professionals who frequently analyze dense technical documentation.
* **The Pain Point:** Parsing extensive, 20+ page research papers to extract specific methodologies or mathematical models is a highly manual process with a massive cognitive load. Existing AI summarizers often provide generic, high-level overviews that fail to address the user's immediate, specific study needs.
* **The Solution & Workflow Goal:** This workflow introduces a dynamic, goal-oriented knowledge extraction pipeline. Instead of generating a static summary, the system ingests a research paper alongside a specific **User-Defined Learning Goal** (e.g., "Extract only the mathematical proofs related to consensus mechanisms"). The system autonomously reads the document, extracts *only* the concepts relevant to that explicit goal, verifies the accuracy of the extraction to prevent hallucinations, and generates a structured pedagogical study guide.
* **Expected Output:** A comprehensive, intelligently formatted Notion page containing a customized study guide tailored strictly to the user's initial learning objective.

## 2. Problem-to-Workflow Breakdown & Node Mapping
To effectively solve this problem, the workflow is decomposed into a multi-step pipeline featuring distinct agentic roles: Dynamic Ingestion, Iterative RAG (Retrieval-Augmented Generation) Extraction, Quality Control, and Goal-Oriented Synthesis. 

The following table details the strict input-output mapping for every node in the pipeline.

| Phase | Node Name | Agent Role / Type | Input | Output | Core Process |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Ingestion** | `Student Form Submission` | Trigger | User form: `paperURL` + `learningGoal` | JSON payload of URL and goal | Captures the explicit user learning directive and the data source document. |
| **Ingestion** | `Fetch Paper PDF` | Deterministic | `paperURL` | Binary PDF Data | Executes an HTTP GET request to download the target document. |
| **Ingestion** | `Extract from File` | Deterministic | Binary PDF Data | Raw text string (`text`) | Parses the binary file into machine-readable text. |
| **Extraction** | `Paper Chunk Loop` & `Create Text Chunk` | Deterministic | Complete raw `text` string | Sequential `textChunk` fragments | Safely chunks the document to prevent AI context-window overflow during processing. |
| **Extraction** | `Information Extractor AI` | **Specialized Extractor** | `textChunk` + `learningGoal` | Extracted concepts (JSON array) | Scans the text strictly through the lens of the learning goal for relevant methodologies and math. |
| **Extraction** | `Formatting Guardrail` | Deterministic | Extracted concepts array | Cleaned JSON data | Enforces strict schema adherence before iterating the loop. |
| **Quality Control** | `Aggregate All Topics` | Deterministic | Array of all loop iterations | Consolidated JSON list | Merges fragmented extraction data into a single, cohesive payload. |
| **Quality Control** | `Critic AI` | **Autonomous Reviewer** | Consolidated topics + `learningGoal` | Approved JSON payload | Cross-references the extracted data against the original goal to filter hallucinations and irrelevant data. |
| **Quality Control** | `Merge Approval Paths` | Deterministic | Critic's decision output | Approved data stream | Structural routing node that passes verified data forward. |
| **Synthesis** | `Lesson Planner AI` | **Synthesizer** | Approved data + `learningGoal` | Long-form Markdown text | Applies pedagogical structuring to generate a cohesive study guide. |
| **Publishing** | `Code in JavaScript` | Deterministic Algorithm| Markdown study guide | `notionBlocks` JSON array | Dynamically chunks text (≤1900 chars) at paragraph breaks to respect API rate limits without breaking sentences. |
| **Publishing** | `Create Notion Study Guide` | Integration | `notionBlocks` JSON array | Live Notion Page | Authenticated API POST request to generate the final workspace artifact. |

## 3. Use of AI vs. Deterministic Steps
This system enforces a strict separation of concerns to maximize reliability, scalability, and output quality.

* **AI Reasoning Steps:** AI is utilized exclusively for cognitive tasks. The `Information Extractor AI` exercises reasoning to find goal-relevant concepts within dense text. The `Critic AI` exercises judgment to review the data against the initial prompt. The `Lesson Planner AI` exercises creativity to synthesize the final educational material.
* **Deterministic Control Logic:** Standard nodes and custom code handle all operational infrastructure. This includes fetching files via HTTP requests, iterating through text via the `Paper Chunk Loop`, enforcing formatting boundaries, aggregating arrays, and using custom JavaScript to parse and shape the final JSON arrays for the Notion API integration.

## 4. Agentic System Diagram
The following flowchart illustrates the complete data pipeline, demonstrating routing, loops, and agentic handoffs.

```mermaid
graph TD
    A[Trigger: User Form<br/>Input: PDF URL + Learning Goal] -->|JSON| B(Fetch Paper PDF)
    B -->|Binary Data| C(Extract from File: PDF to Text)
    C -->|Raw Text| D{Paper Chunk Loop}
    
    D -->|Current textChunk| E[Create Text Chunk]
    E --> F((Agent 1: Information Extractor AI<br/>Guided by Learning Goal))
    F --> G[Formatting Guardrail]
    G -->|Iterate| D
    
    D -->|Loop Complete| H[Aggregate All Topics]
    H --> I((Agent 2: Critic AI<br/>Goal Verification))
    I -->|Approved Data| J[Merge Approval Paths]
    J --> K((Agent 3: Lesson Planner AI<br/>Pedagogical Synthesis))
    K -->|Markdown Text| L[JavaScript Code<br/>Safe API Chunking Algorithm]
    L -->|notionBlocks Array| M[Notion Integration<br/>Publish Final Study Guide]
```
## 6. Current Limitations & Future Improvements

While the current workflow successfully implements an autonomous multi-agent RAG pipeline, real-world testing has highlighted a few architectural limitations that will be addressed in future iterations.

### Current Challenges
1. **Loss of Multi-Modal Data (Tables & Diagrams):** Technical research papers heavily rely on complex tables, architectural diagrams, and visual charts. The current deterministic `Extract from File` node strictly parses raw text, meaning critical visual data is stripped out before the AI agents can analyze it.
2. **Lack of Source Traceability:** The `Information Extractor AI` successfully pulls complex mathematical and algorithmic concepts, but it does not map these extractions back to the exact page numbers or section headers of the original PDF, making deep academic verification slightly difficult for the end-user.
3. **Sequential Processing Bottleneck:** The `Paper Chunk Loop` processes document segments sequentially. For highly dense technical whitepapers (50+ pages), this linear extraction significantly increases the total workflow execution time and memory load.

### Future Architectural Roadmap
To evolve this system from a functioning prototype to a robust, production-ready product, the following upgrades are planned:

* **Multi-Modal Parsing Integration:** We plan to replace the basic text extractor with an advanced OCR and Vision-LLM integration. This will allow the pipeline to interpret complex data tables and translate architectural diagrams into structured markdown, ensuring zero data loss during ingestion.
* **Granular Citation Engine:** The extraction schema will be updated to automatically capture chunk indices. This will allow the system to append exact source citations (e.g., *[Ref: Page 14, Section 3.1]*) to every mathematical formula or algorithm listed in the final study guide.
* **Parallel Chunk Processing:** We plan to upgrade the RAG ingestion pipeline to process text chunks asynchronously/in parallel using sub-workflows. This will drastically reduce latency, allowing the agents to quickly parse massive documents without timing out.
