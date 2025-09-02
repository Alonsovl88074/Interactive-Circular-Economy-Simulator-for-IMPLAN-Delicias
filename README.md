# Interactive Circular Economy Simulator for IMPLAN Delicias

## Executive Summary

This project showcases an interactive web-based simulator designed to generate tailored Circular Economy strategies for businesses in Delicias, Chihuahua. Developed for the Municipal Planning Institute (IMPLAN), this tool leverages a combination of AI, vector databases, and a responsive web interface to empower local businesses with actionable insights for sustainable development. It demonstrates a robust application of Fullstack development principles, integrating a dynamic frontend with a powerful AI-driven backend and efficient data indexing.

## Key Features

*   **AI-Powered Strategy Generation:** Provides detailed, structured proposals for Circular Economy implementation, including benefits, specific activities, involved actors, estimated costs, and local collaboration opportunities.
*   **Context-Aware Responses:** Utilizes a Retrieval-Augmented Generation (RAG) approach to ground AI responses in a comprehensive local knowledge base, ensuring relevance to Delicias, Chihuahua.
*   **Dynamic and Responsive User Interface:** An intuitive web form allows users to input business details, receiving a custom executive summary in a clean, readable format.
*   **Automated Email Notifications:** Sends a copy of each generated proposal to IMPLAN's planning department, facilitating follow-up and data collection.
*   **Modular and Scalable Architecture:** Designed with clear separation of concerns across frontend, backend, and data indexing components.

## Technologies and Applied Skills

This project demonstrates a wide range of skills across the Fullstack development spectrum:

### Frontend Development

*   **HTML5 & CSS3:** Structured semantic content and modern, responsive styling.
*   **JavaScript (ES6+):** Client-side logic for dynamic form interactions, real-time validation, and asynchronous communication with the backend API.
*   **Materialize CSS:** Utilized for a clean, modern, and responsive UI/UX, including form elements, navigation, and interactive components.
*   **jQuery:** Streamlined DOM manipulation, event handling, and AJAX requests, enhancing development efficiency.
*   **AJAX:** Implemented for seamless background communication with the Flask backend, preventing page reloads during AI processing.
*   **UI/UX Design:** Focus on creating an intuitive and accessible interface for business users to easily input information and interpret AI-generated proposals.
*   **DOM Manipulation & Content Post-Processing:** Developed a custom JavaScript function to transform raw AI text output into beautifully formatted HTML, ensuring readability and consistency, including custom bullet points and indentation.

<details>
<summary><strong>Code Snippet: Frontend JavaScript for AI Output Formatting (`simulador-ia-econodel.html`)</strong></summary>

````javascript
// Function to format the AI's plain text output for better display in HTML
function formatAiOutput(text) {
    const lines = text.split('\n').filter(line => line.trim() !== ''); // Filter out truly empty lines
    let finalHtml = '';
    let ulStack = []; // To manage nested <ul> elements

    lines.forEach(line => {
        const lineIndent = line.search(/\S|$/); // Get actual indentation
        const trimmedLine = line.trim();

        // Main Title
        if (trimmedLine.startsWith('Propuesta de Economía Circular para ')) {
            finalHtml += `<h2>${trimmedLine}</h2>`;
            ulStack = []; 
            return;
        }
        // Section Titles (e.g., "1. Descripción General")
        if (/^\d+\.\s/.test(trimmedLine)) {
            while (ulStack.length > 0) { finalHtml += '</li></ul>'; ulStack.pop(); }
            finalHtml += `<h3>${trimmedLine}</h3>`;
            return;
        }
        // Strategy Titles (e.g., "- Estrategia A:")
        if (trimmedLine.startsWith('- Estrategia')) {
            while (ulStack.length > 0) { finalHtml += '</li></ul>'; ulStack.pop(); }
            finalHtml += `<h4>${trimmedLine.substring(trimmedLine.indexOf('- ') + 2)}</h4>`;
            return;
        }
        // List Items (lines starting with '-')
        if (trimmedLine.startsWith('- ')) {
            const liContent = trimmedLine.substring(trimmedLine.indexOf('- ') + 2);
            let displayContent = liContent;
            const activityMatch = liContent.match(/^(Actividad \d+:)\s*(.*)/);
            if (activityMatch) {
                displayContent = `<strong>${activityMatch}</strong> ${activityMatch}`; // Changed to match[1] and match[2]
            }

            let targetLevel = (lineIndent - trimmedLine.indexOf('-')) / 2;
            if (targetLevel < 0) targetLevel = 0; 

            while (ulStack.length > targetLevel) { finalHtml += '</li></ul>'; ulStack.pop(); }
            while (ulStack.length < targetLevel) { finalHtml += '<ul>'; ulStack.push('</li>'); }
            
            finalHtml += `<li>${displayContent}`;
            if (ulStack.length === targetLevel) { ulStack[ulStack.length - 1] = '</li>'; }
            return;
        }
        // Regular paragraphs
        while (ulStack.length > 0) { finalHtml += '</li></ul>'; ulStack.pop(); }
        finalHtml += `<p>${trimmedLine}</p>`;
    });

    while (ulStack.length > 0) { finalHtml += '</li></ul>'; ulStack.pop(); }
    return finalHtml;
}
</details>

Backend Development
Python & Flask: Developed a lightweight RESTful API using Flask to handle incoming requests from the frontend, process data, and interact with the AI model and vector database.
Google Generative AI (Gemini-2.5-flash-lite): Integrated the Gemini model to power the core intelligence of the simulator, generating creative and structured Circular Economy proposals.
Chroma DB: Implemented as a persistent vector database for storing and retrieving contextual information from a localized knowledge base, enabling Retrieval-Augmented Generation (RAG).
Prompt Engineering: Crafted sophisticated prompts to guide the Gemini model, ensuring structured, contextually relevant, and detailed outputs tailored to the user's input and specific local needs of Delicias, adhering to a precise plain-text output format.
Email Automation (smtplib): Configured an email notification system to send generated proposals to a designated IMPLAN email address, enhancing operational efficiency and data archiving.
CORS Handling: Managed Cross-Origin Resource Sharing to allow secure communication between the frontend (served from a different origin) and the Flask API.
Code Snippet: Backend Flask API and Prompt Engineering (chatbot_backend.py)
code
Python

<details>
<summary><strong>Code Snippet: Data Indexer Configuration and Text Splitting (`indexer.py`)</strong></summary>
code
Python
import os
import PyPDF2
import pandas as pd
import json
import chromadb
from chromadb.utils import embedding_functions
from langchain.text_splitter import RecursiveCharacterTextSplitter 

print("Iniciando el proceso de indexación...")

# --- 1. CONFIGURATION ---
CHROMA_DATA_PATH = "chroma_data/"
COLLECTION_NAME = "implan_delicias_docs"
API_KEY = "AIzaSyAXAP1H6CUsd1zozbkrRNAY09WrUi3qUmg" 

# --- INITIALIZE TEXT SPLITTER ---
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,  # Size of each chunk in characters
    chunk_overlap=150, # Overlap to maintain context across chunks
    length_function=len,
)

# ... (file loading functions)

# --- 2. PREPARE CHROMA DB ---
client = chromadb.PersistentClient(path=CHROMA_DATA_PATH)
google_ef = embedding_functions.GoogleGenerativeAiEmbeddingFunction(api_key=API_KEY)

if COLLECTION_NAME in [c.name for c in client.list_collections()]:
    client.delete_collection(name=COLLECTION_NAME)
    print(f"Colección '{COLLECTION_NAME}' antigua eliminada.")

collection = client.create_collection(name=COLLECTION_NAME, embedding_function=google_ef)
print(f"Colección '{COLLECTION_NAME}' nueva creada.")

# --- 3. PROCESS DOCUMENTS WITH CHUNKING AND BATCHING ---
def process_and_store_documents(folder_path='knowledge_base', batch_size=100):
    all_chunks = []
    all_metadatas = []
    
    print("Phase 1: Reading and chunking all documents...")
    for root, _, files in os.walk(folder_path):
        for file in files:
            file_path = os.path.join(root, file)
            loader = None
            if file.endswith('.pdf'): loader = load_text_from_pdf
            elif file.endswith('.csv'): loader = load_text_from_csv
            elif file.endswith('.txt'): loader = load_text_from_txt
            elif file.endswith('.json'): loader = load_text_from_json
            
            if loader:
                for content, metadata in loader(file_path):
                    if content and content.strip():
                        chunks = text_splitter.split_text(content)
                        all_chunks.extend(chunks)
                        all_metadatas.extend([metadata] * len(chunks))

    if not all_chunks:
        print("No se encontraron fragmentos de texto válidos para indexar.")
        return

    print(f"\nPhase 2: Indexing {len(all_chunks)} fragments into the vector database...")
    ids = [str(i) for i in range(len(all_chunks))]
    
    for i in range(0, len(all_chunks), batch_size):
        batch_chunks = all_chunks[i:i + batch_size]
        batch_metadatas = all_metadatas[i:i + batch_size]
        batch_ids = ids[i:i + batch_size]
        
        print(f"  - Processing batch {i//batch_size + 1} of { -(-len(all_chunks)//batch_size) } (fragments {i+1} to {i+len(batch_ids)})...")
        try:
            collection.add(documents=batch_chunks, metadatas=batch_metadatas, ids=batch_ids)
        except Exception as e:
            print(f"    - ERROR: Failed to process this batch. Error: {e}")
            continue
            
    print("\nIndexing completed!")

if __name__ == "__main__":
    process_and_store_documents()

Code Snippet: Data Indexer Configuration and Text Splitting (`indexer.py`)</strong></summary>

import os
import PyPDF2
import pandas as pd
import json
import chromadb
from chromadb.utils import embedding_functions
from langchain.text_splitter import RecursiveCharacterTextSplitter 

print("Iniciando el proceso de indexación...")

# --- 1. CONFIGURATION ---
CHROMA_DATA_PATH = "chroma_data/"
COLLECTION_NAME = "implan_delicias_docs"
API_KEY = "AIzaSyAXAP1H6CUsd1zozbkrRNAY09WrUi3qUmg" 

# --- INITIALIZE TEXT SPLITTER ---
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,  # Size of each chunk in characters
    chunk_overlap=150, # Overlap to maintain context across chunks
    length_function=len,
)

# ... (file loading functions)

# --- 2. PREPARE CHROMA DB ---
client = chromadb.PersistentClient(path=CHROMA_DATA_PATH)
google_ef = embedding_functions.GoogleGenerativeAiEmbeddingFunction(api_key=API_KEY)

if COLLECTION_NAME in [c.name for c in client.list_collections()]:
    client.delete_collection(name=COLLECTION_NAME)
    print(f"Colección '{COLLECTION_NAME}' antigua eliminada.")

collection = client.create_collection(name=COLLECTION_NAME, embedding_function=google_ef)
print(f"Colección '{COLLECTION_NAME}' nueva creada.")

# --- 3. PROCESS DOCUMENTS WITH CHUNKING AND BATCHING ---
def process_and_store_documents(folder_path='knowledge_base', batch_size=100):
    all_chunks = []
    all_metadatas = []
    
    print("Phase 1: Reading and chunking all documents...")
    for root, _, files in os.walk(folder_path):
        for file in files:
            file_path = os.path.join(root, file)
            loader = None
            if file.endswith('.pdf'): loader = load_text_from_pdf
            elif file.endswith('.csv'): loader = load_text_from_csv
            elif file.endswith('.txt'): loader = load_text_from_txt
            elif file.endswith('.json'): loader = load_text_from_json
            
            if loader:
                for content, metadata in loader(file_path):
                    if content and content.strip():
                        chunks = text_splitter.split_text(content)
                        all_chunks.extend(chunks)
                        all_metadatas.extend([metadata] * len(chunks))

    if not all_chunks:
        print("No se encontraron fragmentos de texto válidos para indexar.")
        return

    print(f"\nPhase 2: Indexing {len(all_chunks)} fragments into the vector database...")
    ids = [str(i) for i in range(len(all_chunks))]
    
    for i in range(0, len(all_chunks), batch_size):
        batch_chunks = all_chunks[i:i + batch_size]
        batch_metadatas = all_metadatas[i:i + batch_size]
        batch_ids = ids[i:i + batch_size]
        
        print(f"  - Processing batch {i//batch_size + 1} of { -(-len(all_chunks)//batch_size) } (fragments {i+1} to {i+len(batch_ids)})...")
        try:
            collection.add(documents=batch_chunks, metadatas=batch_metadatas, ids=batch_ids)
        except Exception as e:
            print(f"    - ERROR: Failed to process this batch. Error: {e}")
            continue
            
    print("\nIndexing completed!")

if __name__ == "__main__":
    process_and_store_documents()


Python
## --- 1. CONFIGURATION ---
CHROMA_DATA_PATH = "chroma_data/"
COLLECTION_NAME = "implan_delicias_docs"
API_KEY = "AIzaSyAXAP1H6CUsd1zozbkrRNAY09WrUi3qUmg" 

## --- INITIALIZE TEXT SPLITTER ---
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,  # Size of each chunk in characters
    chunk_overlap=150, # Overlap to maintain context across chunks
    length_function=len,
)

## --- 3. PROCESS DOCUMENTS WITH CHUNKING AND BATCHING ---
def process_and_store_documents(folder_path='knowledge_base', batch_size=100):
    all_chunks = []
    all_metadatas = []
    
    print("Phase 1: Reading and chunking all documents...")
    for root, _, files in os.walk(folder_path):
        for file in files:
            file_path = os.path.join(root, file)
            # ... (file loading logic - omitted for brevity in README)
            if loader:
                for content, metadata in loader(file_path):
                    if content and content.strip():
                        # Divide content into smaller fragments
                        chunks = text_splitter.split_text(content)
                        all_chunks.extend(chunks)
                        all_metadatas.extend([metadata] * len(chunks))

    # ... (rest of batch processing and adding to collection logic - omitted for brevity)
## How to Run
Clone the repository:
code
Bash
git clone [your-repo-url]
cd [your-repo-directory]
Install Python dependencies:
Create a requirements.txt file with the following content:
code
Code
flask
flask-cors
chromadb
google-generativeai
PyPDF2
pandas
langchain
smtplib
Then run:
code
Bash
pip install -r requirements.txt

### Configure API Key and Email Credentials:
Open chatbot_backend.py and replace placeholder values for API_KEY, SENDER_EMAIL, SENDER_PASSWORD, RECEIVER_EMAIL, SMTP_SERVER, and SMTP_PORT with your actual credentials. For SENDER_PASSWORD with Gmail, it is highly recommended to use an App Password generated from your Google account security settings.
Also update API_KEY in indexer.py.

### Prepare your Knowledge Base:
Place relevant .pdf, .csv, .txt, and .json documents containing information about Delicias, Circular Economy principles, local businesses, regulations, etc., into the knowledge_base/ directory. The quality of AI responses heavily depends on the richness of this data.
Index the documents:
code
Bash
python indexer.py
This will create the chroma_data/ directory and populate your vector database.
Start the Flask backend server:
code
Bash
python chatbot_backend.py

### Open the frontend:
Open simulador-ia-econodel.html directly in your web browser.

## Future Enhancements (Roadmap)
External Web Search Integration: Implement Function Calling in the AI model to perform real-time web searches for the most up-to-date local business collaborations or incentives not present in the static knowledge base.

## User Authentication & Profiles: Allow users to save their business profiles and past generated proposals for future reference and iteration.
Admin Dashboard: Develop a dedicated interface for IMPLAN staff to efficiently manage the knowledge base, review generated proposals, and analyze usage statistics for project impact assessment.
Interactive Cost/Benefit Analysis: Integrate more dynamic financial modeling capabilities into the proposals, allowing users to adjust parameters and see estimated returns.
Multilingual Support: Expand the simulator to support other languages relevant to the region or user base.
Advanced Data Visualization: Incorporate charts and graphs into the executive summary for clearer presentation of key metrics and impacts.
