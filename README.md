graph TD
    %% 定义样式
    classDef clinical fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef omics fill:#f3e5f5,stroke:#4a148c,stroke-width:2px;
    classDef cell fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px;
    classDef output fill:#fff3e0,stroke:#e65100,stroke-width:2px,stroke-dasharray: 5 5;

    %% --- 第一阶段：临床试验 ---
    subgraph Phase_I
        direction TB
        Recruit:::clinical --> Random((随机化分组)):::clinical
        Random --> Group_Tx[序贯治疗组]:::clinical
        Random --> Group_Ctrl[对照组/安慰剂]:::clinical

        subgraph Sequential_Logic [序贯给药逻辑]
            Group_Tx -->|Day 1-5 (急性期)| Tx_Stage1[CHEP方: 清热化痰]:::clinical
            Tx_Stage1 -->|Day 6 (转换期)| Tx_Switch{证候演变<br>痰热 -> 气虚}:::clinical
            Tx_Switch -->|Day 6-15 (恢复期)| Tx_Stage2:::clinical
        end

        Group_Ctrl -->|全程| Placebo[安慰剂对照]:::clinical

        subgraph Sampling [纵向生物样本采集]
            S_T0:::clinical
            S_T1:::clinical
            S_T2:::clinical
        end

        Recruit -.-> S_T0
        Tx_Stage1 -.-> S_T1
        Tx_Stage2 -.-> S_T2
        S_T0 & S_T1 & S_T2 --> Biobank:::clinical
    end

    %% --- 第二阶段：多组学分析 ---
    subgraph Phase_II
        direction TB
        Biobank -->|巢式病例对照筛选<br>极好应答 vs 无应答 (n=30+30)| NCC_Selection[样本筛选: Nested Case-Control]:::omics
        
        NCC_Selection --> Meth_Seq:::omics
        NCC_Selection --> RNA_Seq:::omics

        subgraph Analysis_Pipeline [生物信息分析流程]
            Meth_Seq & RNA_Seq --> LMM:::omics
            LMM --> Interaction_Feat[识别治疗响应性时间特征]:::omics
            
            Meth_Seq & RNA_Seq --> MOFA[MOFA+ 多组学因子分析]:::omics
            MOFA --> Latent_Factors[潜在因子识别]:::omics
            Latent_Factors -.->|Pearson相关| Clinical_Data:::omics
            
            Interaction_Feat & Latent_Factors --> Cis_Reg:::omics
            Cis_Reg --> Neg_Corr[筛选负相关配对:<br>Promoter Meth ↓ - Gene Expr ↑]:::omics
        end

        Neg_Corr --> Targets:::output
    end

    %% --- 第三阶段：细胞验证 ---
    subgraph Phase_III [第三阶段：时序性细胞机制验证 (Cellular Validation)]
        direction TB
        Targets --> Cell_Model:::cell
        
        subgraph In_Vitro_Simulation [体外序贯给药模拟]
            Cell_Model -->|OGD 4-6h (急性期)| Step1_Acute[CHEP含药血清处理]:::cell
            Step1_Acute -->|复氧即刻 (Washout)| Step2_Wash:::cell
            Step2_Wash -->|Reperfusion 24-48h (恢复期)| Step3_Recov:::cell
        end

        Step3_Recov --> Molecular_Assay[分子机制检测]:::cell
        Molecular_Assay --> Check_NLRP3:::cell
        Molecular_Assay --> Check_BDNF:::cell
        
        Check_BDNF --> Epigenetic_Intervention:::cell
    end

    Phase_I --> Phase_II
    Phase_II --> Phase_III
    Phase_III -->|机制回馈| Final_Conclusion[结论: 揭示序贯治疗的<br>表观遗传学时空机制]:::output
