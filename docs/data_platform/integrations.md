
``` mermaid
flowchart LR
    %% Nodes
    subgraph SourceSystems [Source Systems]
        A["CRM Systems"]
        B["ERP Systems"]
        C["Marketing Platforms"]
        D["On Prem Apps"]
    end

    subgraph IntegrationLayer [Integration Layer]
        E["API Gateway"]
        F["Message Broker"]
        G["Transformation Worker"]
        H["Persistance Database"]
        I["Sender Worker"]
    end

    subgraph DestinationSystems [Destination Systems]
        J["CRM Systems"]
        K["ERP Systems"]
        L["Marketing Platforms"]
        M["On Prem Apps"]
    end

    %% Edge connections between nodes
    %% From Source Systems to Integration Layer
    A --> E
    B --> E
    C --> E
    D --> E
    F --> G

    %% Within Integration Layer
    E --> F
    E --> G
    G --> |Entity Resolution| H
    H --> I

    %% From Integration Layer to Destination Systems
    I --> J
    I --> K
    I --> L
    I --> M

style E color:#FFFFFF, stroke:#2962FF, fill:#2962FF
style I color:#FFFFFF, stroke:#2962FF, fill:#2962FF

```