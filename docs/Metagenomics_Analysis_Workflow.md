# Metagenomic Analysis Workflow



This diagram shows the complete processing pipeline, combining \*\*quality control, assembly, taxonomic analysis, functional annotation, and visualisation\*\*.



```mermaid

flowchart TD

&nbsp; %% -------- Preprocessing \& Assembly --------

&nbsp; subgraph Preprocessing\["Preprocessing \& Quality Control"]

&nbsp;   direction TB

&nbsp;   P1\["Input: Raw Sequence Reads"]

&nbsp;   P2\["Filter reads (readfq, bowtie2)"]

&nbsp;   P3\["Remove host sequences"]

&nbsp;   P4\["Output: High-quality filtered reads"]

&nbsp; end



&nbsp; subgraph Assembly\["Assembly \& Gene Prediction"]

&nbsp;  direction TB

&nbsp;  A1\["Assemble reads into contigs (MEGAHIT)"]

&nbsp;  A2\["Predict genes (MetaGeneMark)"]

&nbsp;  A3\["Cluster genes into catalog (CD-HIT)"]

&nbsp;  A4\["Map reads back (Bowtie2)"]

&nbsp;  A5\["Output: Gene catalog \& abundance table"]

&nbsp; end



&nbsp; %% -------- Taxonomic Analysis --------

&nbsp; subgraph Taxonomic\["Taxonomic Analysis"]

&nbsp;  direction TB

&nbsp;  T1\["Species annotation (DIAMOND, MEGAN, KRONA)"]

&nbsp;  T2\["Alpha Diversity (Chao1, Shannon, Simpson, Fisher)"]

&nbsp;  T3\["Beta Diversity (Bray-Curtis, NMDS, PERMANOVA)"]

&nbsp;  T4\["Differential abundance (Maaslin2)"]

&nbsp;  T5\["Output: Taxonomic composition \& significant taxa"]

&nbsp; end



&nbsp; %% -------- Functional Annotation --------

&nbsp; subgraph Functional\["Functional Annotation \& Pathways"]

&nbsp; direction TB

&nbsp; V1\["Plots: heatmaps, barplots, ordinations"]

&nbsp; V2\["Krona, ggplot2, pheatmap"]

&nbsp; V3\["Output: Final comprehensive report"]

&nbsp; end



%% -------- Connections --------

P1 --> P2 --> P3 --> P4

P4 --> A1 --> A2 --> A3 --> A4 --> A5

A5 --> T1

T1 --> T2 --> T3 --> T4 --> T5

A5--> F1 --> F2 --> F3 --> F4 --> F5 --> F6

T5 --> V1

F6 --> V1

V1 --> V2 --> V3



&nbsp; %% Highlight key outputs

classDef output fill:#fff2cc,stroke:#000,stroke-width:2px,font-weight:bold;

class P4,A5,T5,F6,V3 output;



&nbsp; %% Color subgraphs

&nbsp; style Preprocessing fill:%dfe7fd,stroke:#003f91,stroke-width:2px

&nbsp; style Assembly fill:#fde7df,stroke:#cc5500,stroke-width:2px

&nbsp; style Taxonomic fill:#fcefed,stroke:#8b0000,stroke-width:2px

&nbsp; style Functional fill:#fef9e7,stroke:#8a6d3b,stroke-width:2px

&nbsp; style Visualisation fill:#e2eafc,stroke:#34568b,stroke-width:2px


```
