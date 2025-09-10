flowchart LR
    %% Orchestrator core
    subgraph Orchestrator[WXO Orchestrate]
        direction TB
        receive([1. Receive Prospect <br/>(web / form / API)])
        startRPA1([2. Start RPA1: <br/>fetch DBD / BOL])
        enrich([3. Data Enrichment <br/>(RAG + Extract)])
        calc([4. Calculate Score <br/>(code/Excel API)])
        gather([5. Gather Internal/External signals <br/>(RAG)])
        approve([6. User Approve / Decision])
        update([7. RPA2: Update <br/>Smart App DB])
        notify([8. Notify / Audit / Logging])
    end

    %% Data stores & services
    subgraph Storage
        ProspectDB[(Prospect DB)]
        ObjectStore[[Object Storage <br/>(pdfs, scraped html)]]
        VectorDB[(Vector DB)]
        ERP[(Internal ERP DB)]
        SmartAppDB[(Smart App DB)]
        ExcelFile[[Excel file (shared)]]
    end

    subgraph AIStack
        EmbeddingSvc[(Embedding service / model)]
        LLM[(watsonx LLM)]
    end

    %% --- Flow ---
    %% Main Orchestrator Sequence
    receive --> startRPA1 --> enrich --> calc --> gather --> approve --> update --> notify

    %% Data Interactions
    receive --"Save Prospect"--> ProspectDB
    startRPA1 --"Store files & data"--> ObjectStore & ProspectDB & ExcelFile
    
    ObjectStore --"Vectorize"--> EmbeddingSvc --> VectorDB
    
    %% Enrichment & Calculation
    enrich --"Uses RAG"--> LLM
    VectorDB --"Provides Context"--> LLM
    ExcelFile --"Input"--> enrich
    enrich --"Updates DB"--> ProspectDB
    LLM --"Output for Scoring"--> calc
    
    %% Signal Gathering & Approval
    gather --"Uses RAG"--> LLM
    ERP --"Internal Signals"--> gather
    ObjectStore --"External Signals"--> gather
    LLM --"Summary for User"--> approve
    
    %% Final Updates
    update --"Updates App DB"--> SmartAppDB
