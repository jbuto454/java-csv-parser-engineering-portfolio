# System Architecture: Philadelphia Data Analysis Pipeline

## High-Level Architecture
The system follows a **three-layer architecture**:
1. **Data Layer** (`datamanagement/`): Handles all I/O (CSV files, API calls)
2. **Processing Layer** (`processor/`): Business logic with memoization and strategies
3. **Presentation Layer** (`ui/`): CLI interface and result formatting

## Component Diagram
