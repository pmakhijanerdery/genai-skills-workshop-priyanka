## Alaska Department of Snow - Online Agent

### Architecture Diagram

```
                                    End User
                                   (Browser)
                                       |
                                   HTTPS Request
                                       |
                                       v
                    +------------------------------------------+
                    |         Cloud Run Service                |
                    |  snow-dept-agent-2kssu2xdxa-uc.a.run.app|
                    |                                          |
                    |    +--------------------------------+    |
                    |    |   Flask Web Application        |    |
                    |    |                                |    |
                    |    |   Frontend     Chat Endpoint   |    |
                    |    |   (HTML/JS)    (/chat POST)    |    |
                    |    +--------------------------------+    |
                    +------------------------------------------+
                                       |
                    +------------------+------------------+
                    |                                     |
                    v                                     v
        +------------------------+          +-------------------------+
        |   Security Layer       |          |   Agent Core Logic      |
        |                        |          |                         |
        | - Input filtering      |          | - Query routing         |
        | - Prompt injection     |          | - RAG or Weather API?   |
        |   detection            |          |                         |
        | - Length validation    |          +-------------------------+
        +------------------------+                       |
                                         +---------------+---------------+
                                         |                               |
                                         v                               v
                          +-------------------------+      +-------------------------+
                          |   RAG Pipeline          |      |   Weather API           |
                          |   (FAQ Queries)         |      |   Integration           |
                          |                         |      |                         |
                          | 1. Query embedding      |      | Weather.gov API         |
                          | 2. Vector search        |      | (5 Alaska cities)       |
                          | 3. Context retrieval    |      |                         |
                          | 4. LLM generation       |      | - Anchorage             |
                          |                         |      | - Fairbanks             |
                          +------------+------------+      | - Juneau                |
                                       |                   | - Sitka                 |
                                       |                   | - Ketchikan             |
                                       |                   +-------------------------+
                    +------------------+------------------+
                    |                                     |
                    v                                     v
        +---------------------------+      +--------------------------------+
        |   Vertex AI Services      |      |   BigQuery Dataset             |
        |                           |      |   alaska_snow_dept             |
        | - Gemini 2.0 Flash Exp    |      |                                |
        |   (Text Generation)       |      |   Table: faqs_embedded         |
        |                           |      |   - 50 FAQ entries             |
        | - Text Embedding 005      |      |   - Vector embeddings          |
        |   (768 dimensions)        |      |     (768 dim per entry)        |
        +---------------------------+      +--------------------------------+
                    |
                    v
        +---------------------------+
        |   Cloud Logging           |
        |                           |
        |   Logger: snow-dept-agent |
        |   - All interactions      |
        |   - Error tracking        |
        |   - Audit trail           |
        +---------------------------+
```

### Data Flow

#### 1. User Query Processing
```
User Query --> Security Filtering --> Valid?
                                       |
                        +--------------+--------------+
                        |                             |
                       No                            Yes
                        |                             |
                Return Error                    Route to Handler
```

#### 2. Weather Query Path
```
User Query --> Detect Keywords (weather, forecast, snow)
           --> Extract City Name
           --> Call Weather.gov API
           --> Return Forecast
```

#### 3. FAQ Query Path
```
User Query --> Generate Query Embedding
           --> Search faqs_embedded Table (Cosine Similarity)
           --> Retrieve Top 3 Matches
           --> Build Context
           --> Send to Gemini
           --> Generate Response
```

#### 4. Logging
```
All Interactions --> Cloud Logging (Audit Trail)
```

### Component Details

#### Security Layer
- Input filtering for prompt injection attempts
- Length validation (max 500 characters)
- Content filtering for harmful patterns
- All blocked attempts logged

#### RAG Pipeline
- Vector similarity search using cosine distance
- Top-3 context retrieval from BigQuery
- Embedding dimension: 768
- Model: Text Embedding 005

#### Weather API Integration
- Real-time data from Weather.gov
- Supports 5 major Alaska cities
- 3-day forecast with temperature and conditions
- Fallback to FAQ system for unsupported cities

#### Vertex AI Services
- Primary Model: Gemini 2.0 Flash Exp
- Embedding Model: Text Embedding 005
- Safety settings: BLOCK_MEDIUM_AND_ABOVE
- Filters: harassment, hate speech, dangerous content

#### BigQuery Dataset
- Dataset: alaska_snow_dept
- Table: faqs_embedded
- 50 FAQ entries with 768-dimensional embeddings
- Indexed for fast vector similarity search

#### Cloud Logging
- Logger name: snow-dept-agent
- Logs all user interactions
- Tracks success/error status
- Provides audit trail for compliance

### Technology Stack

**Backend Framework:** Flask 3.0.0  
**Deployment Platform:** Google Cloud Run  
**AI/ML Services:** Vertex AI (Gemini + Text Embedding)  
**Database:** BigQuery  
**Monitoring:** Cloud Logging  
**External APIs:** Weather.gov  

**Key Dependencies:**
- google-cloud-aiplatform 1.70.0
- google-cloud-bigquery 3.25.0
- google-cloud-logging 3.11.0
- numpy 1.26.4
- flask 3.0.0
- gunicorn 21.2.0
