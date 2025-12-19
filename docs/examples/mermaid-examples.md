# Mermaid Chart Examples

This page demonstrates various types of Mermaid diagrams that you can use in your documentation.

## Flowchart Diagrams

Flowcharts are great for showing processes and decision flows:

```mermaid
graph TD
    A[Start Process] --> B{Check Condition}
    B -->|Yes| C[Action 1]
    B -->|No| D[Action 2]
    C --> E[End]
    D --> E
```

## Sequence Diagrams

Sequence diagrams show interactions between different components:

```mermaid
sequenceDiagram
    participant User
    participant System
    participant Database
    
    User->>System: Submit Request
    System->>Database: Query Data
    Database-->>System: Return Results
    System-->>User: Display Response
```

## Class Diagrams

Class diagrams illustrate object-oriented relationships:

```mermaid
classDiagram
    class Order {
        +String orderNumber
        +Date orderDate
        +calculateTotal()
        +validateOrder()
    }
    class Customer {
        +String customerId
        +String name
        +getCreditLimit()
    }
    class OrderDetail {
        +String productCode
        +int quantity
        +calculateLineTotal()
    }
    
    Order "1" --> "*" OrderDetail
    Customer "1" --> "*" Order
```

## State Diagrams

State diagrams show state transitions:

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Submitted: Submit
    Submitted --> Approved: Approve
    Submitted --> Rejected: Reject
    Approved --> [*]
    Rejected --> Draft: Revise
```

## Entity Relationship Diagrams

ER diagrams show database relationships:

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_DETAIL : contains
    PRODUCT ||--o{ ORDER_DETAIL : "ordered in"
    
    CUSTOMER {
        string customerId PK
        string name
        string email
    }
    ORDER {
        string orderNumber PK
        date orderDate
        string customerId FK
    }
    ORDER_DETAIL {
        string orderNumber FK
        string productCode FK
        int quantity
    }
    PRODUCT {
        string productCode PK
        string description
        decimal price
    }
```

## Gantt Charts

Gantt charts show project timelines:

```mermaid
gantt
    title Project Timeline
    dateFormat YYYY-MM-DD
    section Phase 1
    Requirements Gathering    :2024-01-01, 10d
    Design                    :2024-01-11, 15d
    section Phase 2
    Development               :2024-01-26, 20d
    Testing                   :2024-02-15, 10d
    section Phase 3
    Deployment                :2024-02-25, 5d
```

## Pie Charts

Pie charts show proportional data:

```mermaid
pie title Order Status Distribution
    "Pending" : 25
    "Approved" : 45
    "Shipped" : 20
    "Cancelled" : 10
```

## Git Graph

Git graphs show version control branches:

```mermaid
gitgraph
    commit id: "Initial"
    branch develop
    checkout develop
    commit id: "Feature A"
    commit id: "Feature B"
    checkout main
    merge develop
    commit id: "Release 1.0"
```

## Tips for Using Mermaid

1. **Syntax**: Always use triple backticks with `mermaid` language identifier
2. **Node IDs**: Use camelCase or underscores for node IDs (no spaces)
3. **Labels**: Wrap labels with special characters in quotes
4. **Testing**: Preview your diagrams locally before committing
5. **Complexity**: Keep diagrams simple and focused for better readability

