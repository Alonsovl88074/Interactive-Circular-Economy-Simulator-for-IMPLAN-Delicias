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

<strong>Code Snippet: Frontend JavaScript for AI Output Formatting (`simulador-ia-econodel.html`)
    
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
                displayContent = `<strong>${activityMatch}</strong> ${activityMatch}`;
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
}````

### Backend Development

*   **Python & Flask:** Developed a lightweight RESTful API using Flask to handle incoming requests from the frontend, process data, and interact with the AI model and vector database.
*   **Google Generative AI (Gemini-2.5-flash-lite):** Integrated the Gemini model to power the core intelligence of the simulator, generating creative and structured Circular Economy proposals.
*   **Chroma DB:** Implemented as a persistent vector database for storing and retrieving contextual information from a localized knowledge base, enabling Retrieval-Augmented Generation (RAG).
*   **Prompt Engineering:** Crafted sophisticated prompts to guide the Gemini model, ensuring structured, contextually relevant, and detailed outputs tailored to the user's input and specific local needs of Delicias, adhering to a precise plain-text output format.
*   **Email Automation (`smtplib`):** Configured an email notification system to send generated proposals to a designated IMPLAN email address, enhancing operational efficiency and data archiving.
*   **CORS Handling:** Managed Cross-Origin Resource Sharing to allow secure communication between the frontend (served from a different origin) and the Flask API.

<details>
<summary><strong>Code Snippet: Backend Flask API and Prompt Engineering (`chatbot_backend.py`)</strong></summary>

````python
# Function to create a highly structured prompt for the AI
def create_structured_prompt(user_data, context_chunks):
    """Generates a detailed prompt for the AI to produce a structured Circular Economy proposal."""
    
    context = "\n\n".join(context_chunks)
    user_info_summary = f"""
    Información de la Empresa/Contacto:
    - Nombre: {user_data.get('nombre_empresa')}
    - Sector: {user_data.get('sector_actividad')}
    - Giro: {user_data.get('giro_empresa')}
    - Empleados: {user_data.get('numero_empleados')}
    - Ubicación en Delicias: {user_data.get('ubicacion_empresa')}
    - Productos/Servicios: {user_data.get('productos_servicios')}
    - Recursos Consumidos: {user_data.get('recursos_consumidos')}
    - Desechos Producidos: {user_data.get('tipos_desechos')}
    - Prácticas Actuales: {user_data.get('practicas_actuales_residuos')}
    - Estrategias de Interés: {user_data.get('estrategias_interes') or 'No especificado'}
    - Motivación Principal: {user_data.get('motivacion')}
    """

    prompt = f"""
    You are 'Implani', an expert AI assistant of the Municipal Planning Institute (IMPLAN) of Delicias, Chihuahua.
    Your goal is to generate a detailed and structured Circular Economy proposal for a company in Delicias, Chihuahua, based on the user's information and available context.

    **GOLDEN RULE:** Your primary source of truth is the CONTEXT and USER INFORMATION below. If specific local information for Delicias is not in the context, provide general but relevant suggestions.

    **RELEVANT IMPLAN DELICIAS CONTEXT:**
    {context}

    **USER INFORMATION:**
    {user_info_summary}

    **RESPONSE GENERATION & FORMATTING RULES (PLAIN TEXT):**
    1.  **Strictly Structured Format:** Generate the response in the following plain text format. Do not use Markdown characters (like ##, ***) for titles. Do not use asterisks for bolding.
    2.  **Concise Line Spacing:** Minimize line breaks. Use only one line break between paragraphs and list items, and two line breaks between main sections.
    3.  **Dash Bullet Points:** Always use a dash (-) followed by a space for list items. For sub-lists or nested activities, use two additional spaces per level for indentation before the dash.
    4.  **Local Relevance:** Prioritize integrating Delicias-specific information. If specific local businesses related to user's waste are found in CONTEXT, name them. Otherwise, suggest *types* of businesses.
    5.  **Creativity & Detail:** Be creative and detailed in strategies and activities, adapting them to the user's input.
    6.  **Practical Focus:** Proposals must be actionable and realistic for a business in Delicias.
    7.  **Tone:** Professional, friendly, and proactive.

    ---
    Propuesta de Economía Circular para "{user_data.get('nombre_empresa')}" en Delicias, Chihuahua

    1. Descripción General de la Estrategia
       [Briefly describe the overall vision of the proposed circular economy strategy, considering the user's sector and motivation.]

    2. Beneficios Esperados
       - [Detail the key benefits the company could gain (e.g., cost reduction, environmental impact, improved image, innovation, new business opportunities).]

    3. Estrategias y Actividades Específicas
       - Estrategia A: [Strategy Name, e.g., Industrial Symbiosis for Organic Waste]
         - Actividad 1: [Activity details, e.g., Identify local companies in Delicias interested in composting organic waste from their process.]
         - Actividad 2: [Activity details, e.g., Establish a collection or delivery agreement with nearby farmers in Delicias.]
       - Estrategia B: [Strategy Name, e.g., Water Resource Optimization]
         - Actividad 1: [Activity details, e.g., Audit water usage at the plant and identify leaks or inefficient use.]
         - Actividad 2: [Activity details, e.g., Implement rainwater harvesting systems or greywater treatment and reuse.]
    
    4. Actores Involucrados
       - [Identify key internal and external actors (e.g., company personnel, IMPLAN Delicias, municipal government, universities, other local businesses, suppliers, customers).]

    5. Cronograma Sugerido (Fases)
       - Fase 1: Diagnosis and Research (e.g., 1-3 months)
       - Fase 2: Pilot Implementation (e.g., 3-6 months)
       - Fase 3: Scaling and Monitoring (e.g., 6-12 months)

    6. Costos Estimados (General Range)
       - [Provide a general range of potential costs (e.g., initial technology investment, training, permits, operating costs) and classify them as low, medium, or high. Mention the possibility of seeking local/state support or incentives in Delicias.]

    7. Oportunidades de Colaboración Local en Delicias
       - [Type of Business 1 or business name if found in context, e.g., local farms 'La Huerta', Recycling Plant 'Delicias Ecológica' or simply "Local farms"]
       - [Type of Business 2 or business name, e.g., Plastic recyclers, Metal collection centers]
       - [Type of Institution 1, e.g., Universidad Tecnológica de Delicias for research and development]
       - If no specific information is found, you can add: "It is recommended to conduct local stakeholder mapping to identify potential partners."

    8. Indicadores de Seguimiento (KPIs)
       - [Propose 2-3 key indicators to measure the success and impact of the implemented strategies (e.g., % waste reduction, water savings, increase in recycled material use, cost reduction).]
    ---
    """
    return prompt

@app.route('/chat', methods=['POST'])
def chat():
    try:
        # ... (logging and data extraction)

        user_data_from_form = data.get("user_data") 
        # ... (validation)
        
        search_query = f"""
        Empresa: {user_data_from_form.get('nombre_empresa')}
        Sector: {user_data_from_form.get('sector_actividad')}
        Giro: {user_data_from_form.get('giro_empresa')}
        Desechos: {user_data_from_form.get('tipos_desechos')}
        Interés: {user_data_from_form.get('estrategias_interes')}
        Ubicación: Delicias, Chihuahua
        """

        results = collection.query(
            query_texts=[search_query],
            n_results=7 
        )
        context_chunks = results['documents'][0]
        
        prompt = create_structured_prompt(user_data_from_form, context_chunks)
        response = model.generate_content(prompt)
        ai_generated_answer = response.text

        # ... (email sending logic)
        
        return jsonify({"answer": ai_generated_answer})
    except Exception as e:
        # ... (error handling)
        pass


Data Indexing & Management
Python (PyPDF2, pandas, langchain): Developed scripts to ingest and process unstructured data from various sources (PDF, CSV, TXT, JSON) into a structured format suitable for the vector database.
Langchain (RecursiveCharacterTextSplitter): Utilized for intelligent text chunking, ensuring that relevant pieces of information are maintained together while optimizing for vector database embedding and retrieval efficiency.
Chroma DB: Managed the vector database lifecycle, including collection creation, deletion, and adding documents with associated metadata.
Embedding Functions (GoogleGenerativeAiEmbeddingFunction): Leveraged Google's embedding models to convert text chunks into numerical vectors, enabling semantic search capabilities.
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

# ... (file loading functions - omitted for brevity)

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

</details>
How to Run
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
(Note: smtplib is a built-in Python module, no need to list it in requirements.txt.)
Then run:
code
Bash
pip install -r requirements.txt
Configure API Key and Email Credentials:
Open chatbot_backend.py and replace placeholder values for API_KEY, SENDER_EMAIL, SENDER_PASSWORD, RECEIVER_EMAIL, SMTP_SERVER, and SMTP_PORT with your actual credentials. For SENDER_PASSWORD with Gmail, it is highly recommended to use an App Password generated from your Google account security settings.
Also update API_KEY in indexer.py.
Prepare your Knowledge Base:
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
Open the frontend:
Open simulador-ia-econodel.html directly in your web browser.


## Future Enhancements (Roadmap)
External Web Search Integration: Implement Function Calling in the AI model to perform real-time web searches for the most up-to-date local business collaborations or incentives not present in the static knowledge base.
User Authentication & Profiles: Allow users to save their business profiles and past generated proposals for future reference and iteration.
Admin Dashboard: Develop a dedicated interface for IMPLAN staff to efficiently manage the knowledge base, review generated proposals, and analyze usage statistics for project impact assessment.
Interactive Cost/Benefit Analysis: Integrate more dynamic financial modeling capabilities into the proposals, allowing users to adjust parameters and see estimated returns.
Multilingual Support: Expand the simulator to support other languages relevant to the region or user base.
Advanced Data Visualization: Incorporate charts and graphs into the executive summary for clearer presentation of key metrics and impacts.
