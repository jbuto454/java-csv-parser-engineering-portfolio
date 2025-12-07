# System Architecture: Philadelphia Data Analysis Pipeline

## High-Level Architecture

The system follows a **three-layer architecture**:

1. **Data Layer** (`datamanagement/`): Handles all I/O operations including CSV parsing, JSON parsing, and API integration
2. **Processing Layer** (`processor/`): Contains business logic with memoization caching and strategy pattern implementations
3. **Presentation Layer** (`ui/`): CLI interface for user interaction and result formatting

---

## Component Diagram

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#4A90A4', 'primaryTextColor': '#fff', 'primaryBorderColor': '#2D5A6A', 'lineColor': '#5C6BC0', 'secondaryColor': '#81C784', 'tertiaryColor': '#FFB74D'}}}%%

flowchart TB
    subgraph Entry["🚀 Entry Point"]
        Main["Main.java"]
    end

    subgraph Presentation["📺 Presentation Layer"]
        UI["UserInterface"]
    end

    subgraph Processing["⚙️ Processing Layer"]
        direction TB
        Processor["Processor<br/><i>memoization cache</i>"]
        
        subgraph Strategy["Strategy Pattern"]
            direction LR
            AS["«interface»<br/>AveragingStrategy"]
            SPS["SalesPriceStrategy"]
            NBS["NumberOfBedroomsStrategy"]
        end
        
        AS -.->|implements| SPS
        AS -.->|implements| NBS
    end

    subgraph DataLayer["📂 Data Layer"]
        direction TB
        
        subgraph Parsers["CSV Parsing"]
            direction TB
            SuperCSV["«abstract»<br/>SuperCSVParser&lt;T&gt;<br/><i>buffered reading</i><br/><i>header indexing</i>"]
            PopReader["PopulationDataReader"]
            PropReader["PropertyDataReader"]
            SRCSVReader["ServiceRequestsCSVReader"]
        end
        
        subgraph Factory["Factory Pattern"]
            SRFactory["ServiceRequestsReaderFactory"]
            SRInterface["«interface»<br/>ServiceRequestsDataReader"]
            SRJSONReader["ServiceRequestsJSONReader"]
        end
        
        subgraph APIIntegration["API Integration"]
            SRRepo["ServiceRequestsRepository"]
            APIClient["APIClient<br/><i>HTTP client</i>"]
            FileMgr["FileManager"]
        end
        
        SuperCSV -.->|extends| PopReader
        SuperCSV -.->|extends| PropReader
        SuperCSV -.->|extends| SRCSVReader
        
        SRInterface -.->|implements| SRCSVReader
        SRInterface -.->|implements| SRJSONReader
        
        SRFactory -->|creates| SRInterface
        SRRepo --> APIClient
        SRRepo --> FileMgr
    end

    subgraph Models["📊 Data Models"]
        direction LR
        PopRecord["PopulationRecord"]
        PropData["PropertyData"]
        SRData["ServiceRequestData"]
    end

    subgraph Utils["🔧 Utilities"]
        direction LR
        NumUtils["numericUtils<br/><i>validation</i>"]
        TimeUtils["TimestampUtils<br/><i>date formatting</i>"]
    end

    subgraph Logging["📝 Logging"]
        Logger["Logger<br/><i>«Singleton»</i>"]
    end

    subgraph External["☁️ External"]
        API[("Philadelphia<br/>Open Data API")]
        CSVFiles[("CSV Files")]
        JSONFiles[("JSON Files")]
    end

    %% Main connections
    Main --> UI
    Main --> Processor
    Main --> SRFactory
    Main --> SRRepo
    Main --> PopReader
    Main --> PropReader
    Main --> Logger

    %% UI connections
    UI --> Processor
    UI --> Logger
    UI -->|injects| AS

    %% Processor connections
    Processor --> Models
    Processor --> AS

    %% Data layer connections
    PopReader --> PopRecord
    PropReader --> PropData
    SRCSVReader --> SRData
    SRJSONReader --> SRData
    
    %% Parser utilities
    PopReader --> NumUtils
    PropReader --> NumUtils
    SRCSVReader --> TimeUtils
    SRJSONReader --> TimeUtils

    %% External connections
    APIClient <-->|HTTP| API
    FileMgr --> JSONFiles
    SuperCSV --> CSVFiles
    SRJSONReader --> JSONFiles
```

---

## Data Flow Diagram

This simplified view shows how data moves through the pipeline from ingestion to output:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#2196F3', 'lineColor': '#607D8B'}}}%%

flowchart LR
    subgraph Sources["Data Sources"]
        CSV["📄 CSV Files"]
        API["☁️ Philly API"]
    end

    subgraph Ingestion["Data Ingestion"]
        direction TB
        Factory["Factory<br/>Pattern"]
        Parser["SuperCSVParser<br/><i>buffered I/O</i><br/><i>header indexing</i>"]
        APIClient["APIClient"]
    end

    subgraph Models["Data Models"]
        direction TB
        Pop["Population<br/>Records"]
        Prop["Property<br/>Data"]
        SR["Service<br/>Requests"]
    end

    subgraph Processing["Processing Engine"]
        direction TB
        Proc["Processor"]
        Cache["Memoization<br/>Cache"]
        Strat["Strategy<br/>Pattern"]
    end

    subgraph Output["Output"]
        UI["CLI<br/>Interface"]
        Log["Logger<br/>«Singleton»"]
    end

    CSV --> Parser
    API --> APIClient
    APIClient --> Factory
    Parser --> Factory
    Factory --> Models
    
    Models --> Proc
    Proc <--> Cache
    Proc <--> Strat
    
    Proc --> UI
    UI --> Log

    style Sources fill:#E3F2FD,stroke:#1976D2
    style Ingestion fill:#E8F5E9,stroke:#388E3C
    style Models fill:#FFF3E0,stroke:#F57C00
    style Processing fill:#F3E5F5,stroke:#7B1FA2
    style Output fill:#FFEBEE,stroke:#C62828
```

---

## Design Patterns Implementation

The project implements three classic Gang of Four design patterns:

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Singleton["🔒 Singleton Pattern: Logger"]
        direction TB
        S1["Private Constructor<br/><code>private Logger()</code>"]
        S2["Static Instance<br/><code>private static final Logger instance</code>"]
        S3["Global Access Point<br/><code>Logger.getInstance()</code>"]
        
        S1 --> S2 --> S3
        
        S4["Benefits:<br/>• Single log file<br/>• No resource conflicts<br/>• Consistent state"]
    end

    subgraph Factory["🏭 Factory Pattern: ServiceRequestsReaderFactory"]
        direction TB
        F1["Client Code<br/><code>Main.java</code>"]
        F2["Factory<br/><code>ServiceRequestsReaderFactory.createReader(filename)</code>"]
        F3["«interface»<br/>ServiceRequestsDataReader"]
        F4["CSVReader"]
        F5["JSONReader"]
        
        F1 -->|"requests reader"| F2
        F2 -->|".csv extension"| F4
        F2 -->|".json extension"| F5
        F3 -.->|implements| F4
        F3 -.->|implements| F5
        
        F6["Benefits:<br/>• Decoupled creation<br/>• Easy to extend<br/>• Single responsibility"]
    end

    subgraph Strategy["🎯 Strategy Pattern: AveragingStrategy"]
        direction TB
        ST1["Context<br/><code>Processor</code>"]
        ST2["«interface»<br/>AveragingStrategy<br/><code>getValue(PropertyData)</code>"]
        ST3["SalesPriceStrategy<br/><code>return property.getSalesPrice()</code>"]
        ST4["NumberOfBedroomsStrategy<br/><code>return property.getNumberOfBedrooms()</code>"]
        
        ST1 -->|"uses"| ST2
        ST2 -.->|implements| ST3
        ST2 -.->|implements| ST4
        
        ST5["Benefits:<br/>• Open/Closed Principle<br/>• Runtime flexibility<br/>• Eliminates conditionals"]
    end

    style Singleton fill:#FFF9C4,stroke:#F9A825
    style Factory fill:#C8E6C9,stroke:#388E3C
    style Strategy fill:#E1BEE7,stroke:#7B1FA2
```

---

## Class Diagram

Detailed UML class diagram showing inheritance hierarchies, interface implementations, and key relationships:

```mermaid
%%{init: {'theme': 'base'}}%%

classDiagram
    direction TB
    
    %% CSV Parser Hierarchy
    class SuperCSVParser~T~ {
        <<abstract>>
        -BufferedReader reader
        -String[] headersArray
        -Map~String, Integer~ headerIndexMap
        -String[] currentRow
        -char[] buffer
        -StringBuilder fieldBuilder
        +parseNext() Optional~T~
        #getObject()* T
        #getObjectFields(T obj)*
        #getUsedColumns()* Set~String~
        #getDefaultValueForClass()* String
        #getField(header, row) String
        -parseNextRow() List~String~
        -processHeaders()
    }
    
    class PopulationDataReader {
        +readPopulationData() List~PopulationRecord~
        #getUsedColumns() Set~String~
        #getObject() PopulationRecord
    }
    
    class PropertyDataReader {
        +readPropertyData() Map~String, List~PropertyData~~
        #getUsedColumns() Set~String~
        #getObject() PropertyData
    }
    
    class ServiceRequestsCSVReader {
        +readServiceRequestsData() Map
        #getUsedColumns() Set~String~
        #getObject() ServiceRequestData
    }
    
    SuperCSVParser <|-- PopulationDataReader
    SuperCSVParser <|-- PropertyDataReader
    SuperCSVParser <|-- ServiceRequestsCSVReader

    %% Service Requests Interface & Factory
    class ServiceRequestsDataReader {
        <<interface>>
        +readServiceRequestsData() Map
        +validateObjectDataFormat()
        +createServiceRequestObject()
    }
    
    class ServiceRequestsJSONReader {
        -String filename
        +readServiceRequestsData() Map
        #getObject() ServiceRequestData
    }
    
    class ServiceRequestsReaderFactory {
        <<factory>>
        +createReader(filename)$ ServiceRequestsDataReader
    }
    
    ServiceRequestsDataReader <|.. ServiceRequestsCSVReader
    ServiceRequestsDataReader <|.. ServiceRequestsJSONReader
    ServiceRequestsReaderFactory ..> ServiceRequestsDataReader : creates

    %% Strategy Pattern
    class AveragingStrategy {
        <<interface>>
        +getValue(PropertyData) Double
    }
    
    class SalesPriceStrategy {
        +getValue(PropertyData) Double
    }
    
    class NumberOfBedroomsStrategy {
        +getValue(PropertyData) Double
    }
    
    AveragingStrategy <|.. SalesPriceStrategy
    AveragingStrategy <|.. NumberOfBedroomsStrategy

    %% Processor with Memoization
    class Processor {
        -Map serviceRequestMap
        -Map propertyDataMap
        -List populationDataList
        -Integer totalPopulation
        -Map~String, Integer~ populationPerZipCache
        -Map~String, Map~ serviceRequestPerCapitaCache
        -Map~String, Integer~ avgSalesPriceCache
        -Map~String, Integer~ avgNumberOfBedroomsCache
        +getTotalPopulation() int
        +getPopulationFromZip(zipCode) int
        +getServiceRequestsPerCapita(date) Map
        +getAverageForZip(zipCode, strategy) int
        +getTotalSalesPricePerCapita(zipCode) int
        +resetAllCaches()
    }
    
    Processor --> AveragingStrategy : uses

    %% Singleton Logger
    class Logger {
        <<singleton>>
        -PrintWriter pw
        -boolean isUsingSystemErr
        -Logger instance$
        -Logger()
        +getInstance()$ Logger
        +setDestination(filepath) boolean
        +log(message)
        +close()
        +reset()
    }

    %% Data Models
    class PopulationRecord {
        -Integer population
        -String zipCode
        -boolean valid
        +getZipCode() String
        +getPopulation() Integer
        +isValid() boolean
    }
    
    class PropertyData {
        -Double salesPrice
        -Double numberOfBedrooms
        -String zipCode
        -boolean valid
        +getZipCode() String
        +getSalesPrice() Double
        +getNumberOfBedrooms() Double
    }
    
    class ServiceRequestData {
        -String zipCode
        -String serviceName
        -String serviceNotice
        -String requestedDate
        -String closedDate
        -boolean valid
    }

    %% API Integration
    class APIClient {
        -HttpClient httpClient
        -String baseUrl
        +executeQuery(query) String
    }
    
    class ServiceRequestsRepository {
        -APIClient apiClient
        -String CARTO_API_URL$
        +downloadAllToFile(filePath)
        -getAllServiceRequests() String
    }
    
    ServiceRequestsRepository --> APIClient : uses
    
    %% User Interface
    class UserInterface {
        -Processor processor
        -Scanner in
        -boolean propertiesFileProvided
        -boolean populationFileProvided
        -Logger logger
        +start()
        +displayMenu()
    }
    
    UserInterface --> Processor : uses
    UserInterface --> Logger : uses
    UserInterface --> AveragingStrategy : injects

    %% Relationships to data models
    PopulationDataReader ..> PopulationRecord : creates
    PropertyDataReader ..> PropertyData : creates
    ServiceRequestsCSVReader ..> ServiceRequestData : creates
    ServiceRequestsJSONReader ..> ServiceRequestData : creates
```

---

## Layer Responsibilities

| Layer | Package | Responsibilities |
|-------|---------|------------------|
| **Presentation** | `ui/` | User interaction, menu display, input validation, output formatting |
| **Processing** | `processor/` | Business logic, calculations, memoization caching, strategy execution |
| **Data Access** | `datamanagement/` | File I/O, CSV/JSON parsing, API communication, data transformation |
| **Utilities** | `util/` | Data models (POJOs), validation helpers, timestamp formatting |
| **Cross-cutting** | `logging/` | Application-wide logging (Singleton) |

---

## Key Architectural Decisions

### 1. Generic Abstract Parser (`SuperCSVParser<T>`)
- **Decision**: Use generics with template method pattern
- **Benefit**: Single parsing logic handles multiple data types
- **Trade-off**: Slightly more complex base class, but eliminates code duplication

### 2. HashMap for Header Index Mapping
- **Decision**: `HashMap<String, Integer>` instead of linear search
- **Benefit**: O(1) column lookup vs O(n) per field access
- **Impact**: Critical for large files with many columns

### 3. Memoization in Processor
- **Decision**: Cache calculation results by key (zipcode, date)
- **Benefit**: Repeated queries return instantly
- **Trade-off**: Memory usage increases with unique queries

### 4. Factory for Reader Selection
- **Decision**: Runtime polymorphism based on file extension
- **Benefit**: Client code doesn't need to know concrete implementations
- **Extensibility**: Adding XML support requires only new reader + factory case
