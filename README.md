---
config:
  layout: fixed
---
flowchart LR
 subgraph Quality_Control["Quality_Control"]
        QC1["Filter Reads using readfq, bowtie2"]
        QC2["Remove Host Genome Sequences"]
  end
 subgraph Preprocessing["Preprocessing"]
    direction TB
        PP1["Input: Raw Sequence Reads"]
        Quality_Control
        PP4["Output: High-quality Filtered Reads"]
  end
 subgraph Assembly["Assembly"]
    direction TB
        A1["Tool: MEGAHIT"]
        A2["Assemble Reads into Contigs"]
        A3["Output: Contigs"]
  end
 subgraph Gene_Prediction["Gene_Prediction"]
        GP1["Predict Genes"]
        GP2["Core-pan Genome Analysis"]
  end
 subgraph Gene_Analysis["Gene_Analysis"]
    direction TB
        GA1["Tools: MetaGeneMark, CD-HIT, Bowtie2"]
        Gene_Prediction
        GA4["Output: Gene Catalog and Abundance Data"]
  end
 subgraph Taxonomy_Analysis["Taxonomy_Analysis"]
        TA1["Species Annotation"]
        TA2["Generate Heatmaps, PCA, PCoA"]
  end
 subgraph Taxonomy["Taxonomy"]
    direction TB
        T1["Tools: DIAMOND, MEGAN, KRONA"]
        Taxonomy_Analysis
        T4["Output: Taxonomic Classification"]
  end
 subgraph Functional_Analysis["Functional_Analysis"]
        FN1["Functional Analysis"]
        FN2["Pathway Analysis"]
  end
 subgraph Functionality["Functionality"]
    direction TB
        FA1["Tools: DIAMOND, KEGG, eggNOG, CAZy"]
        Functional_Analysis
        FA4["Output: Functional Insights"]
  end
 subgraph Resistance_Annotation["Resistance_Annotation"]
        RN1["Annotate Resistance Genes"]
        RN2["Visualize Results: Bar Plots, Heatmaps"]
  end
 subgraph Resistance["Resistance"]
    direction TB
        RA1["Tools: RGI, ARDB-CARD"]
        Resistance_Annotation
        RA4["Output: Resistance Profiles"]
  end
 subgraph Reporting["Reporting"]
        RP1["Generate Visualizations"]
        RP2["Compile Final Report"]
  end
 subgraph Visualization_Reporting["Visualization_Reporting"]
    direction TB
        VR1["Tools: Krona, Circos, R"]
        Reporting
        VR4["Output: Comprehensive Report"]
  end
    Preprocessing --> Assembly
    Assembly --> Gene_Analysis
    Gene_Analysis --> Taxonomy
    Taxonomy --> Functionality
    Functionality --> Resistance
    Resistance --> Visualization_Reporting
    PP1 --> Quality_Control
    Quality_Control --> PP4
    A1 --> A2
    A2 --> A3
    GA1 --> Gene_Prediction
    Gene_Prediction --> GA4
    T1 --> Taxonomy_Analysis
    Taxonomy_Analysis --> T4
    FA1 --> Functional_Analysis
    Functional_Analysis --> FA4
    RA1 --> Resistance_Annotation
    Resistance_Annotation --> RA4
    VR1 --> Reporting
    Reporting --> VR4
     PP4:::important
     A3:::important
     GA4:::important
     T4:::important
     FA4:::important
     RA4:::important
     VR4:::important
    classDef important fill:#fff2cc,stroke:#000,stroke-width:2px,font-weight:bold
    style Preprocessing fill:#dfe7fd,stroke:#003f91,stroke-width:2px,color:#000000
    style Assembly fill:#fde7df,stroke:#cc5500,stroke-width:2px
    style Gene_Analysis fill:#e5fadb,stroke:#116530,stroke-width:2px
    style Taxonomy fill:#fcefed,stroke:#8b0000,stroke-width:2px
    style Functionality fill:#fff7e3,stroke:#daa520,stroke-width:2px
    style Resistance fill:#e6f2ff,stroke:#1e90ff,stroke-width:2px
    style Visualization_Reporting fill:#f5f5dc,stroke:#808080,stroke-width:2px
