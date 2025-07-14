```mermaid 
flowchart TB 
    subgraph Annotation["Functional Annotation (eggNOG-mapper)"] 
        A1["Run emapper.py on .faa files"] 
        A2["Output: .annotations, .hits, .seed_orthologs"] 
    end 
    subgraph Parsing["Extract Functional Terms"] 
        P1["Parse .annotations files"] 
        P2["Extract KEGG KOs, COGs, GO, EC"] 
        P3["Generate counts matrices (eggNOG_KEGG_ko_counts_matrix.tsv, etc)"] 
    end 
    subgraph Pathway_Summary["Pathway-level Summary"] 
        S1["Summarise KEGG Pathways per sample"] 
        S2["Output: KEGG_pathway_counts_per_sample.tsv"] 
    end 
    subgraph Statistical_Analysis["Statistical Analysis"] 
        M1["Merge pathway counts with lipid metadata"] 
        C1["Compute correlations (heatmap)"] 
        L1["Linear models for LDL, HDL, Cholesterol"] 
        S3["Significance filtering (p < 0.05)"] 
    end 
    subgraph Visualization["Visualization & Interpretation"] 
        V1["Heatmap: Correlations (KEGG vs lipids)"] 
        V2["Barplots: Significant pathways by lipid trait"] 
        V3["Coefficient heatmap (pathway x trait)"] 
    end 
    %% Flow 
    A1 --
    A2 -- -- --
    P3 -- --
    S2 --
    M1 -- --
``` 
