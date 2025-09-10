```mermaid
flowchart LR
  %% Orchestrator core
  subgraph Orchestrator[WXO Orchestrate]
    direction TB
    receive([1. Receive Prospect (web / form / API)])
    startRPA1([2. Start RPA1: fetch DBD / BOL])
    enrich([3. Data Enrichment (RAG + Extract)])
    calc([4. Calculate Score (code/Excel API)])
    gather([5. Gather Internal/External signals (RAG)])
    approve([6. User Approve / Decision])
    update([7. RPA2: Update Smart App DB])
    notify([Notify / Audit / Logging])
  end

  %% Data stores & services
  subgraph Storage
    ProspectDB[(Prospect DB)]
    ObjectStore[[Object Storage (pdfs, scraped html)]]
    VectorDB[(Vector DB)]
    ERP[(Internal ERP DB)]
    SmartAppDB[(Smart App DB)]
    ExcelFile[[Excel file (shared)]]
  end

  subgraph AIStack
    EmbeddingSvc[(Embedding service / model)]
    LLM[(watsonx LLM)]
  end

  %% Flow
  receive --> ProspectDB
  receive --> startRPA1
  startRPA1 --> ObjectStore
  startRPA1 --> ProspectDB
  startRPA1 --> ExcelFile
  ObjectStore --> EmbeddingSvc --> VectorDB
  ExcelFile --> enrich
  enrich --> VectorDB
  enrich --> ProspectDB
  VectorDB --> LLM
  LLM --> calc
  ERP --> gather
  ObjectStore --> gather
  gather --> LLM
  LLM --> approve
  approve --> update
  update --> SmartAppDB
  update --> notify
