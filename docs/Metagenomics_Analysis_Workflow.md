# 🚀 Metagenomic Analysis Workflow

This diagram shows the complete processing pipeline, combining **quality control, assembly, taxonomic analysis, functional annotation, and visualization**.

```mermaid
flowchart TD
    %% -------- Preprocessing & Assembly --------
    subgraph Preprocessing["Preprocessing & Quality Control"]
        direction TB
        P1["Input: Raw Sequence Reads"]
        P2["Filter reads (readfq, bowtie2)"]
        P3["Remove host sequences"]
        P4["Output: High-quality filtered reads"]
    end

    subgraph Assembly["Assembly & Gene Prediction"]
        direction TB
        A1["Assemble reads into contigs (MEGAHIT)"]
        A2["Predict genes (MetaGeneMark)"]
        A3["Cluster genes into catalog (CD-HIT)"]
        A4["Map reads back (Bowtie2)"]
        A5["Output: Gene catalog & abundance table"]
    end

    %% -------- Taxonomic Analysis --------
    subgraph Taxonomic["Taxonomic Analysis"]
        direction TB
        T1["Species annotation (DIAMOND, MEGAN, KRONA)"]
        T2["Alpha diversity (Chao1, Shannon, Simpson, Fisher)"]
        T3["Beta diversity (Bray-Curtis, NMDS, PERMANOVA)"]
        T4["Differential abundance (Maaslin2)"]
        T5["Output: Taxonomic composition & significant taxa"]
    end

    %% -------- Functional Annotation --------
    subgraph Functional["Functional Annotation & Pathways"]
        direction TB
        F1["Annotate genes (eggNOG-mapper)"]
        F2["Extract KEGG KOs, COGs, GO, EC"]
        F3["Summarise pathways per sample"]
        F4["Merge with lipid metadata"]
        F5["Stat tests: correlations & linear models"]
        F6["Output: Significant pathways (p < 0.05)"]
    end

    %% -------- Visualization & Reporting --------
    subgraph Visualization["Visualization & Reporting"]
        direction TB
        V1["Plots: heatmaps, barplots, ordinations"]
        V2["Krona, Circos, ggplot2, pheatmap"]
        V3["Output: Final comprehensive report"]
    end

    %% -------- Connections --------
    P1 --> P2 --> P3 --> P4
    P4 --> A1 --> A2 --> A3 --> A4 --> A5
    A5 --> T1
    T1 --> T2 --> T3 --> T4 --> T5
    A5 --> F1 --> F2 --> F3 --> F4 --> F5 --> F6
    T5 --> V1
    F6 --> V1
    V1 --> V2 --> V3

    %% Highlight key outputs
    classDef output fill:#fff2cc,stroke:#000,stroke-width:2px,font-weight:bold;
    class P4,A5,T5,F6,V3 output;

    %% Color subgraphs
    style Preprocessing fill:#dfe7fd,stroke:#003f91,stroke-width:2px
    style Assembly fill:#fde7df,stroke:#cc5500,stroke-width:2px
    style Taxonomic fill:#fcefed,stroke:#8b0000,stroke-width:2px
    style Functional fill:#fef9e7,stroke:#8a6d3b,stroke-width:2px
    style Visualization fill:#e2eafc,stroke:#34568b,stroke-width:2px
