## 📚 Tutorial: Ask questions against 2025 IBM financial reports

**Scenario:** You have **IBM financial reports from 2025** (one document per quarter) and a **test_data.json** file with questions about them. The goal is to run a RAG workflow from a notebook in OpenShift AI (using ai4rag-style execution) against a **Llama-stack RAG server**, then explore answers and retrieval results in the notebook.

This tutorial walks you through: creating a project and workbench, preparing S3 with the documents and test data, ensuring the [Llama stack is set up](../../llamastack/SETUP.md) and the RAG stack is deployed, running the AutoRAG notebook, and exploring the results in the notebook.

## Table of contents

- [Create a project and workbench](#create-a-project-and-workbench)
- [Deploy Llama-stack server with RAG stack](#deploy-llama-stack-server-with-rag-stack)
- [Create S3 connection and upload documents](#create-s3-connection-and-upload-documents)
- [Attach S3 connection to the workbench](#attach-s3-connection-to-the-workbench)
- [Open and configure the AutoRAG notebook](#open-and-configure-the-autorag-notebook)
- [Run the notebook and explore results](#run-the-notebook-and-explore-results)

### 🏗️ Create a project and workbench

| Step | Action |
|------|--------|
| **①** | Log in to Red Hat OpenShift AI. |
| **②** | Go to **Data science projects** and create a new project (e.g. `ibm-reports-rag`). |
| **③** | Create a **workbench** (notebook environment) in the project. Choose an image that includes the dependencies required by the AutoRAG notebook (e.g. Python, ai4rag-related libraries if used). For full steps, see [Creating a project and workbench](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.8/html/getting_started_with_red_hat_openshift_ai_self-managed/creating-a-project-workbench_get-started). |

### 🚀 Deploy Llama-stack server with RAG stack

| Step | Action |
|------|--------|
| **①** | In the project, deploy a **Llama-stack server** with the **RAG stack** enabled (chat model, embedding model, vector store such as Milvus). Follow [Llama stack setup](../../llamastack/SETUP.md) for installation and configuration; see also [Deploying a RAG stack in a project](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.0/html/working_with_llama_stack/deploying-a-rag-stack-in-a-project_rag). |
| **②** | Note the RAG/API endpoint and any credentials the notebook will need to call the stack. |

### 📦 Create S3 connection and upload documents

| Step | Action |
|------|--------|
| **①** | In the project, open **Connections** and add an **S3 compatible object storage** connection to a bucket you will use for documents and test data. |
| **②** | Download **IBM financial reports from 2025** from [IBM Financial Reporting](https://www.ibm.com/investor/financial-reporting): under **Find a quarterly earnings presentation**, select year **2025** and download the PDFs for **Q1**, **Q2**, **Q3**, and **Q4** (e.g. Press Release, Charts, or Prepared Remarks per quarter, as needed). |
| **③** | Upload those PDFs to your S3 bucket — one file per quarter (or one combined set). Place them in a known path (e.g. `documents/2025/` or `documents/`). Use the format expected by your RAG stack/notebook. |
| **④** | Upload the **benchmark JSON** (e.g. `benchmark.json` or `test_data.json`) to the same bucket. The file must be a JSON array of objects with `question`, `correct_answers` (array of strings), and `correct_answer_document_ids` (array of document filenames/IDs that contain the answer). Example: `[{"question": "What was IBM's revenue in Q1 2024?", "correct_answers": ["Revenue of $14.5 billion..."], "correct_answer_document_ids": ["ibm-1q24-earnings-press-release.pdf"]}]`. |
| **⑤** | Note the **bucket name** and **object keys** (paths) for the documents and for `test_data.json`; you will set these in the notebook if required. |

### 🔗 Attach S3 connection to the workbench

| Step | Action |
|------|--------|
| **①** | Open **Workbenches**, edit your workbench, and **attach the S3 connection** you created in [Create S3 connection and upload documents](#create-s3-connection-and-upload-documents) so the notebook can read from the bucket. |
| **②** | Save and restart the workbench if prompted. |

### 📓 Open and configure the AutoRAG notebook

| Step | Action |
|------|--------|
| **①** | Upload or clone the **AutoRAG notebook** into the workbench: [run_ai4rag.ipynb](https://github.com/IBM/ai4rag/blob/dev-samples/samples/run_ai4rag.ipynb) (ai4rag `dev-samples` branch). The notebook installs data-processing components from `pipelines-components` (branch `rhoai_autorag_data_processing_pipeline`): test data loader, documents sampling, and text extraction (Docling). |
| **②** | Set **S3 credentials** in the notebook (or via env): `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_ENDPOINT`. Set **bucket name** and **object keys** for the documents prefix and for the benchmark file (e.g. `benchmark.json`). |
| **③** | Set the **Llama-stack client** URL and API key: `LLAMA_STACK_CLIENT_BASE_URL` and `LLAMA_STACK_CLIENT_API_KEY` (or set them in the notebook for `LlamaStackClient(base_url=..., api_key=...)`). Use the URL of the Llama-stack RAG server you deployed in [Deploy Llama-stack server with RAG stack](#deploy-llama-stack-server-with-rag-stack) (see [Llama stack setup](../../llamastack/SETUP.md) if you have not deployed it yet). |
| **④** | Ensure the benchmark JSON format matches: list of objects with `question`, `correct_answers`, and `correct_answer_document_ids`. Extracted documents must have metadata `document_id` matching those IDs (e.g. stem of the filename). |

### ▶️ Run the notebook and explore results

| Step | Action |
|------|--------|
| **①** | Run the notebook **cell by cell** from top to bottom. The notebook will: **Setup** — install `boto3` and KFP data-processing components; **Prepare experiment data** — upload sample PDFs and benchmark JSON to S3; **Process input documents** — test data loader → documents sampling → text extraction (Docling to Markdown); **Run ai4rag experiment** — `LlamaStackClient`, `AI4RAGSearchSpace`, `GAMOptSettings`, `AI4RAGExperiment` (e.g. `vector_store_type="ls_milvus"`); **Review results** — format best evaluation, use `query_rag_pattern` or `interactive_rag_query` for Q&A with grounding documents. |
| **②** | **Explore the results** in the notebook: best configuration, generated answers, retrieved chunks, and evaluation metrics (e.g. faithfulness). Use this to assess RAG behavior and tune documents or test data as needed. |
