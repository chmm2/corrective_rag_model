Advanced RAG with Self-Correction using LangGraph

![alt text](https://img.shields.io/badge/Python-3.10%2B-blue)


![alt text](https://img.shields.io/badge/License-MIT-green)


![alt text](https://img.shields.io/badge/Built%20with-LangChain-purple)

This repository contains an implementation of an advanced, self-correcting Retrieval-Augmented Generation (RAG) system. It leverages the power of LangGraph to create a cyclical, state-driven workflow that can dynamically assess document relevance, fall back to web search for off-topic queries, and even rewrite its own questions to improve retrieval accuracy. The entire system is powered by the blazing-fast inference of Groq with the Llama 3 model.

🚀 The Problem with Simple RAG

Standard RAG pipelines are powerful but can be brittle. They often fail silently in a few common ways:

Irrelevant Query: The user's question has nothing to do with the documents, but the RAG system tries to force an answer anyway.

Poor Retrieval: The vector search returns documents that are not actually helpful for answering the specific question.

Incomplete Answers: The retrieved context contains only part of the answer, leading to a hallucinatory or incomplete response.

This project implements a Corrective RAG (CRAG) architecture to address these issues head-on.

✨ Key Features

Self-Correction Loop: If initial documents are irrelevant, the agent rewrites the user's query and tries again.

Dynamic Routing: The graph first classifies if a question is on-topic. Off-topic questions are routed to a web search tool instead of the document store.

Relevance Grading: Each retrieved document is individually graded for its relevance to the question, ensuring only high-quality context is passed to the generator.

Stateful Agent: Built with LangGraph, the agent maintains a state (AgentState) that tracks the question, documents, grades, and rewrite attempts.

High-Speed Inference: Uses ChatGroq with Llama 3 for near-instantaneous classification, grading, and generation.

Web Search Fallback: Integrates DuckDuckGoSearchRun for questions that are outside the scope of the provided PDF document.

🏗️ Architecture Diagram

The agent's logic is defined as a state graph. This graph dictates the flow of information and the decisions the agent makes at each step.

(You should generate this image from your notebook and save it in your repository)

Workflow Explained:

Topic Decision: The user's query first enters the topic_decision node. A Groq-powered classifier determines if the question is relevant to the general subject matter of the document base.

Routing:

If off-topic, the query is routed to the web_search node, which uses DuckDuckGo to find an answer and then terminates.

If on-topic, the query proceeds to the retrieve_docs node.

Retrieval & Grading:

retrieve_docs fetches relevant document chunks from the Pinecone vector store.

document_grader assesses each retrieved document.

Generation or Rewrite:

The gen_router checks the grades. If any document is marked as "Yes" (relevant), the flow moves to generate_answer.

If all documents are marked "No", the flow moves to rewrite_query.

The rewriter reframes the question for better clarity and specificity, then sends it back to retrieve_docs for another attempt.

Final Answer: The generate_answer node synthesizes the verified, relevant documents into a coherent answer for the user.

🛠️ Tech Stack

Orchestration: LangChain & LangGraph

LLM/Inference: Groq (Llama 3 8B)

Vector Store: Pinecone

Embeddings: HuggingFace (sentence-transformers/all-MiniLM-L6-v2)

Document Loading: PyPDF

Tools: DuckDuckGo Search

⚙️ Setup and Installation

Follow these steps to get the project running on your local machine.

1. Clone the Repository

Generated bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name


2. Create a Virtual Environment
It's highly recommended to use a virtual environment to manage dependencies.

Generated bash
python -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Bash
IGNORE_WHEN_COPYING_END

3. Install Dependencies
Install all required packages from the requirements.txt file.

Generated bash
pip install -r requirements.txt
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Bash
IGNORE_WHEN_COPYING_END

4. Set Up Environment Variables
You will need API keys for Groq and Pinecone.

Create a file named .env in the root of the project.

Copy the contents of .env.example into it and add your keys.

.env.example:

Generated code
GROQ_API_KEY="your_groq_api_key"
PINECONE_API_KEY="your_pinecone_api_key"
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
IGNORE_WHEN_COPYING_END

5. Configure Pinecone
The code is configured to use a Pinecone index named "final-touch". Make sure you have created a serverless index with this name in your Pinecone account. The dimension should match the embeddings model (which is 384 for all-MiniLM-L6-v2).

6. Add Your Document
Place the PDF document you want to query in the root directory and name it new_check.pdf, or update the filename in the notebook.

▶️ How to Run

Make sure your virtual environment is activated.

Start the Jupyter Notebook server:

Generated bash
jupyter notebook
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Bash
IGNORE_WHEN_COPYING_END

Open the crag_implement_updates.ipynb file and run the cells sequentially.

The first run will:

Load the PDF.

Split it into chunks.

Generate embeddings.

Index the documents in your Pinecone vector store.

Subsequent runs will use the already indexed data.

📊 Demonstration: Example Walkthroughs

The notebook output showcases the agent's corrective capabilities.

Example 1: Off-Topic Query → Web Search

When asked a question irrelevant to the PDF content, the agent correctly routes to the web.

Query: "who is the author of this?" (when context is poor)

Path: topic_decision → off_topic → web_search → END

Result: The agent attempts a web search. (Note: The demo shows a rate-limit error, which is a potential issue with the public DuckDuckGo API)

Example 2: Bad Retrieval → Query Rewrite → Success

This is the core of the CRAG pattern. The agent fails, corrects itself, and succeeds.

Initial Query: "Who all are the referneces of this research paper?"

Step 1: The retriever fetches documents, but the document_grader marks them all as "No" because they contain author info, not references.

Step 2: The gen_router sends the state to the rewrite_query node.

Step 3: The LLM rewrites the query to be more precise:

Rewritten Query: "Who are the authors and references cited in this research paper?"

Step 4: The new query is sent back to retrieve_docs. This time, it pulls relevant documents for both authors and references.

Step 5: The document_grader marks the new documents as "Yes".

Step 6: The flow proceeds to generate_answer, providing a complete, correct response.

This demonstrates the system's resilience and ability to overcome the limitations of basic vector search.

📄 License

This project is licensed under the MIT License. See the LICENSE file for details.
