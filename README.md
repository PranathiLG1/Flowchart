flowchart LR

%% =======================
%% STYLES (GitHub Mermaid)
%% =======================
classDef user fill:#1F6FEB,stroke:#0B2E59,stroke-width:2px,color:#ffffff;
classDef ui fill:#1F6FEB,stroke:#0B2E59,stroke-width:2px,color:#ffffff;
classDef mulesoft fill:#2EA043,stroke:#14532D,stroke-width:2px,color:#ffffff;
classDef dw fill:#2EA043,stroke:#14532D,stroke-width:2px,color:#ffffff;
classDef gray fill:#6B7280,stroke:#374151,stroke-width:2px,color:#ffffff;
classDef ext fill:#F59E0B,stroke:#92400E,stroke-width:2px,color:#111827;
classDef decision fill:#FDE68A,stroke:#92400E,stroke-width:2px,color:#111827;
classDef err fill:#4B5563,stroke:#111827,stroke-width:2px,color:#ffffff;

%% =======================
%% LEFT SIDE (USER + UI)
%% =======================
U[User / Customer]:::user
H1[Hybris<br/>(E-commerce UI)<br/>Browse / Cart / Checkout]:::ui

U -->|Step 1: Website open| H1

%% =======================
%% PRICING FLOW (Steps 2–7)
%% =======================
subgraph PF[Pricing Flow (Steps 2–7)]
direction TB

MS1[MuleSoft (Anypoint Platform)<br/>Experience + Process + System APIs]:::mulesoft
DW1[DataWeave<br/>Mapping + Transformation]:::dw
DF[DataFamiling]:::gray
SAPQ{SAP validation required?<br/>(Yes/No)}:::decision
DW2[DataWeave<br/>Mapping + Transformation]:::dw
FP[Step 6: Final Price Response]:::dw

MS1 --> DW1
DW1 --> DF
DF --> SAPQ
SAPQ -->|No| DW2
SAPQ -->|Yes| SAPV
DW2 --> FP
end

H1 -->|Step 2: Fetch pricing + mapping| MS1

%% =======================
%% STATUS UPDATE FLOW (Steps 13–14) - Pricing side
%% =======================
subgraph SU1[Status Update Flow (Steps 13–14)]
direction TB
AXDW[AX Data Warehouse<br/>Pricing + Mapping]:::ext
AX1[Dynamics AX<br/>Create Sales Order]:::ext
SAP1[SAP<br/>Tax / ATP / Shipment / Invoice Status]:::ext
AXDW --> AX1 --> SAP1
end

DW1 -->|Step 3: Pricing data| AXDW
SAPV --> SAP1

%% SAP validation target (kept outside for clean routing)
SAPV[SAP<br/>Validation]:::ext

%% =======================
%% ORDER FLOW (Steps 8–12)
%% =======================
subgraph OF[Order Flow (Steps 8–12)]
direction TB

H2[Hybris<br/>(E-commerce UI)<br/>Browse / Cart / Checkout]:::ui
AX2[Dynamics AX<br/>Create Sales Order]:::mulesoft
V[Step 9: Validate + ID mapping]:::dw
ACK[Step 11: Order# + Ack]:::dw

H2 -->|Step 8: Place Order| V --> AX2
AX2 --> ACK
end

%% Connect from top UI to order UI (same screen / same system)
H1 -.-> H2

%% =======================
%% STATUS UPDATE FLOW (Steps 13–14) - Order side
%% =======================
subgraph SU2[Status Update Flow (Steps 13–14)]
direction TB
SAP2[SAP<br/>Tax / Inventory / Status]:::ext
end

ACK --> SAP2

%% =======================
%% ERROR HANDLING (bottom)
%% =======================
EH[Error Handling<br/>Retry + Alerts + Logs]:::err

%% "dashed influence" lines
MS1 -.-> EH
DW1 -.-> EH
DF  -.-> EH
DW2 -.-> EH
AX1 -.-> EH
SAP1 -.-> EH
AX2 -.-> EH
SAP2 -.-> EH
