# **Slide AI Search App**

## **Overview**

This app template will help you build a multi-modal search service using `GPT-4o` with Metadata Extraction and Vector Index. It uses [Pathway](https://github.com/pathwaycom/llm-app) for indexing and retrieving slides from PowerPoint and PDF presentations.

How is this different?

* Build highly accurate RAG pipelines powered by indexes that are updated in real-time.
* Pathway uses vision language models to understand and index your presentations and PDFs, automatically updating as changes are made.
* Get started with a minimalistic and production-ready approach.

Boost productivity with accurate search across your PowerPoints, PDFs, and Slides all within your work environment. Try out the [demo](https://sales-rag-chat.demo.pathway.com/#search-your-slide-decks) here.


## Quickstart

Check the `.env.example`, create a new `.env` file and fill in the template. 
For a quick start, you need to only change the following fields:
- `PATHWAY_LICENSE_KEY`
- `OPENAI_API_KEY`

This app template is available for free via [Pathway Scale](https://pathway.com/features). Get your [license key here](https://pathway.com/user/license) and fill in the `PATHWAY_LICENSE_KEY` here in the `.env` file.

## How it Helps

**1) Improved Efficiency:**

* **Save Efforts:** You no longer need to manually sift through countless presentations.
* **Faster Information Retrieval:** Instantly find specific information with a few keywords or descriptive prompts, saving you time when preparing for presentations or reviewing past projects.

**2) Enhanced Organization**

* **Automated Categorization:** You can organize your slide library by topic, project, or other criteria. Configure the schema file to cusomize the parsed fields.

**3) Enhanced Reliability**

* **Automatic Updates:** Hybrid indexes update automatically whenever a new slide is added or removed, ensuring your information is always current and accurate.

**4) Automated Slide Parsing:** 

* Process PPTX and PDF slide decks with vision language models to extract the content.

**5) Flexible Data Sources:** 

* Compatible with local directories, SharePoint, Google Drive, and other Pathway connectors, ensuring a wide range of application scenarios can be supported.

By automating the extraction and retrieval of slide information, this app addresses the critical pain point of managing and utilizing extensive slide decks efficiently, enhancing productivity and information accuracy for sales teams.


## Architecture:

The architecture of the Slides AI Search App is designed to connect various local or cloud repositories, transforming and indexing slides for efficient querying. It supports integration with closed and open-source LLMs for enhanced search capabilities.

![Architecture](ai-slides-diagram.svg)

This demo consists of three parts:
* `app.py`: Pathway app that handles parsing, indexing and backend.
* `nginx`: File server that hosts images to be consumed by the UI.
* `UI`: A Streamlit UI for interacting with the app.


## How it works:

### **Data Ingestion**

1. **Data Sources**:
    * The application reads slide files (PPTX and PDF) from a specified directory. The directory is set to `./data/`in the `app.py` file.
    * In the default app setup, the connected folder is a local file folder. You can add more folders and file sources, such as [Google Drive](https://pathway.com/developers/user-guide/connectors/gdrive-connector/#google-drive-connector) or [Sharepoint](https://pathway.com/developers/user-guide/connecting-to-data/connectors/#tutorials), by adding a line of code to the template.
    * More inputs can be added by configuring the `sources` list in the `app.py`.


### **Slide Parsing and Indexing**


1. **Parsing**:
    * The [`SlideParser`](https://pathway.com/developers/api-docs/pathway-xpacks-llm/parsers#pathway.xpacks.llm.parsers.SlideParser) from Pathway is used to parse the slides. The parser is configured to parse a text description and schema that is defined in the `parse_schema.yaml`.
    * Our example schema includes fields such as `category`, `tags`, `title`, `main_color`, `language`, and `has_images`. This can be modified for specific use cases.
    * Note that, UI is configured to make use of two extracted fields `category` and `language`, these need to be kept for the UI to work. However, the app can still be used without the UI with different schemas or no parsed schema.
2. **Embedding**:
    * Parsed slide content is embedded with the OpenAI's `text-embedding-ada-002` embedder.
    * The embeddings are then stored in Pathway's vector store using the `SlidesVectorStoreServer`.
3. **Metadata Handling**:
    * Images and files are dumped into local directories (`storage/pw_dump_images` and `storage/pw_dump_files`).
    * Each slide gets a unique ID with `add_slide_id` function in the `app.py`. This helps with opening files and images from the the UI.
    

### **Query Handling**

1. **Retrieval Augmented Generation (RAG)**:
    * The `DeckRetriever` class builds the backend, handling all steps of the application from parsing files to serving the endpoints. Refer to the [API docs](https://pathway.com/developers/api-docs/pathway-xpacks-llm/question_answering#pathway.xpacks.llm.question_answering.DeckRetriever) for more information.

## Pipeline Organization

This folder contains several components necessary for setting up and running the Sales Slide RAG application:


1. **app.py**:
    * The main application that sets up the slide search functionality. It configures the OpenAI chat model, slide parser, vector store, and initializes the DeckRetriever for handling queries.
2. **parse_schema.yaml**:
    * Defines the schema for parsing the slides and including fields such as `category`, `tags`, `title`, `main_color`, `language`, and `has_images`.
    * These fields will be appended to the `metadata`, if you prefer to also add them to `text` field, set `include_schema_in_text` of `SlideParser` to `True`.
3. **.env**:
    * Config file for the environment variables, such as the OpenAI API key and Pathway key.


## **Prerequisites/Configuration**

### **Environment Setup**

1. **OpenAI API Key**: 
    * Get an API key from the [OpenAI’s API Key Management page](https://platform.openai.com/account/api-keys). Keep this API key secure.
    * Configure your key in the `.env` file.
    * You can refer to the stub file `.env.example` in this repository.
    * Note: This is only needed in OpenAI LLMs and embedders. It is also possible to use other multi-modal, local LLMs and embedders.

2. **Pathway’s License Key**: 
    * This app template is available for free via [Pathway Scale](https://pathway.com/features).
    * Get your [license key here](https://pathway.com/user/license).

3. **SCHEMA_FILE_PATH**:
    * Path to file that defines the schema to be parsed. It can be kept as default and the `parse_schema.yaml` can be configured.

4. **SEARCH_TOPK**:
    * Number of elements to be retrieved from the index by default.

## How to run the project

Make sure you are in the right directory:
```bash
cd examples/pipelines/slides_search
```

### Locally
Running the whole demo without Docker is not suggested as there are three components. 

1. **Download and Install LibreOffice:**
    * Download LibreOffice from the [LibreOffice website](https://www.libreoffice.org/download/download-libreoffice).
    * Follow the installation instructions specific to your operating system. \

2. **Verify LibreOffice Installation:**
    * Download LibreOffice from the LibreOffice website.
    * Open a terminal or command prompt and run the following command:
    * You should see the LibreOffice version information, indicating LibreOffice is installed correctly.

        **Purpose:** LibreOffice helps with converting PPTX files into PDFs, which is essential for the document processing workflow in the Slides AI Search App.

If you are on Windows, please refer to the [running with Docker](#Running-with-docker) section below. 

To run the Pathway app without the UI, 

```bash
python app.py
```

### Running with Docker 

Build the Docker with:

```bash
docker-compose build
```

And, run with:

```bash
docker-compose up
```

This will start all three components of the demo.

## Using the app

After Docker is running, you will see a stream of logs of your files being parsed.

### Accessing the UI

On your browser, visit [`http://localhost:8501`](http://localhost:8501/) to access the UI.

Here, you will see a search bar, some filters, and information about the indexed documents on the left side.


### Sending requests to the server

#### With CURL

UI is not a necessary component, especially for developers. If you are interested in building your own app, check out the following ways to use the app:

First, let's check the indexed files:
```bash
curl -X 'POST'   'http://0.0.0.0:8000/v1/pw_list_documents'   -H 'accept: */*'   -H 'Content-Type: application/json'
```

This will return a list of metadata from the indexed files.

Now, let's search through our slides:


```bash
curl -X 'POST'   'http://0.0.0.0:8000/v1/pw_ai_answer'   -H 'accept: */*'   -H 'Content-Type: application/json'   -d '{
  "prompt": "diagrams that contain value propositions" 
}'
```

This will search through our files, and return parsed slides with the `text`, `slide_id` and other `metadata` (also including the parsed schema).

#### With the Pathway RAG Client

Import RAGClient with:

```python
from pathway.xpacks.llm.question_answering import RAGClient
```

Initialize the client:

```python
# conn = RAGClient(url=f"http://{PATHWAY_HOST}:{PATHWAY_PORT}")

# with the default config
conn = RAGClient(url=f"http://localhost:8000")
```

List the indexed files:
```python
conn.pw_list_documents()
```
> `[{'path': 'data/slide.pdf'}, ...`

Query the app:

```python
conn.pw_ai_answer("introduction slide")
```
> `[{'dist': 0.47761982679367065, 'metadata': ...`


## Not sure how to get started? 

Let's discuss how we can help you build a powerful, customized RAG application. [Reach us here to talk or request a demo!](https://pathway.com/solutions/enterprise-generative-ai?modal=requestdemo)