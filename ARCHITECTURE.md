# MawinguOps System Architecture - As Implemented

## Actual Production Architecture Diagram

```mermaid
graph TD
    %% User Interfaces
    subgraph Users["End Users"]
        Farmers["Farmers with Basic Phones"]
        WebUsers["Web Users and Developers"]
    end
    
    %% External Services Actually Used
    subgraph ExternalAPIs["External Service Layer"]
        OpenMeteo["Open-Meteo Weather API<br/>Real-time forecasts"]
        ATalkingGW["Africas Talking Gateway<br/>USSD Session management"]
    end
    
    %% Google Cloud Platform - What We Actually Built
    subgraph GCP["Google Cloud Platform"]
        subgraph CloudRun["Cloud Run Container Service"]
            ExpressApp["Node.js Express Server<br/>39 REST API Endpoints"]
        end
        
        subgraph Storage["Data Storage"]
            SQLiteDB["SQLite Database<br/>farmer profiles<br/>advisory history"]
            SecretMgr["Secret Manager<br/>API keys<br/>credentials"]
        end
        
        subgraph Monitoring["Observability"]
            CloudLogs["Cloud Logging<br/>Request error tracking"]
            CloudMetrics["Cloud Monitoring<br/>Performance metrics"]
        end
    end
    
    %% Core Application Services Actually Built
    subgraph MawinguCore["MawinguOps Core Application"]
        subgraph APILayer["API Endpoints Layer"]
            USSDWebhook["POST /api/ussd<br/>webhook<br/>Africas Talking callback"]
            AdvisoryAPI["GET/POST /api/advisory<br/>Weather-based recommendations"]
            FarmerAPI["POST /api/register<br/>GET /api/farmers"]
            HealthAPI["GET /api/health<br/>System status"]
        end
        
        subgraph ServiceLayer["Business Logic Services"]
            USSDService["ussdService.js<br/>Menu navigation<br/>Session state management"]
            WeatherService["weatherService.js<br/>Open-Meteo integration<br/>Data caching"]
            AdvisoryEngine["advisoryEngine.js<br/>Crop-specific rules<br/>Decision logic"]
            MLPredictor["mlPredictor.js<br/>Scikit-learn model<br/>Planting predictions"]
            DatabaseService["database.js<br/>SQLite operations<br/>Farmer management"]
        end
        
        subgraph WebInterface["Frontend Layer"]
            WebUI["Static HTML/CSS/JS<br/>Advisory request form"]
            USSDSimulator["USSD Testing Interface<br/>Development tools"]
        end
    end
    
    %% User Flow - USSD Primary
    Farmers --> ATalkingGW
    ATalkingGW --> USSDWebhook
    USSDWebhook --> USSDService
    USSDService --> DatabaseService
    USSDService --> WeatherService
    USSDService --> AdvisoryEngine
    
    %% User Flow - Web Interface
    WebUsers --> WebUI
    WebUI --> AdvisoryAPI
    AdvisoryAPI --> WeatherService
    AdvisoryAPI --> AdvisoryEngine
    
    %% External API Integration
    WeatherService --> OpenMeteo
    AdvisoryEngine --> MLPredictor
    
    %% Data Storage
    DatabaseService --> SQLiteDB
    USSDService --> CloudLogs
    WeatherService --> CloudLogs
    
    %% Response Flow
    USSDService --> ATalkingGW
    ATalkingGW --> Farmers
    AdvisoryAPI --> WebUsers
    
    %% Infrastructure
    ExpressApp --> APILayer
    APILayer --> ServiceLayer
    ServiceLayer --> Storage
    ExpressApp --> Monitoring
    
    %% Styling
    classDef users fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
    classDef external fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef gcp fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef core fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef services fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    classDef api fill:#fce4ec,stroke:#d81b60,stroke-width:2px
    
    class Users,Farmers,WebUsers users
    class ExternalAPIs,OpenMeteo,ATalkingGW external
    class GCP,CloudRun,Storage,Monitoring gcp
    class MawinguCore,WebInterface core
    class ServiceLayer,USSDService,WeatherService,AdvisoryEngine,MLPredictor,DatabaseService services
    class APILayer,USSDWebhook,AdvisoryAPI,FarmerAPI,HealthAPI api
```

## Key Architecture Components

### **Production Infrastructure**
- **Google Cloud Run**: Serverless Node.js containers with auto-scaling 0-100 instances
- **Deployment URL**: `https://mawingu-ops-109647674635.us-central1.run.app`
- **Region**: us-central1 (Iowa, USA - optimal for global access)

### **External Service Integrations**
- **Open-Meteo API**: Free weather data with excellent Kenya coverage
- **Africa's Talking**: USSD gateway with shortcode *384*7460#

### **Core Application Services**
- **39 REST API Endpoints**: Complete API surface for all functionality
- **USSD Service**: Session management and menu navigation
- **Weather Service**: Real-time data fetching and caching
- **Advisory Engine**: Crop-specific recommendation logic
- **ML Predictor**: Scikit-learn model for planting predictions
- **Database Service**: SQLite operations for farmer data

### **Data Flow Architecture**
1. **USSD Flow**: Farmer dials → Africa's Talking → USSD webhook → Processing → Response
2. **Web Flow**: Browser → Static files → API calls → Services → Response
3. **External APIs**: Weather data and ML predictions processed for optimal performance

### **Scalability Features**
- **Auto-scaling**: 0 instances when idle ($0 cost) → up to 100 instances (10,000+ users)
- **Performance**: Sub-200ms response for warm instances, 2-3s cold start
- **Reliability**: Fallback mechanisms for all external service dependencies

### **Monitoring & Observability**
- **Cloud Logging**: All requests, errors, and service interactions
- **Cloud Monitoring**: Performance metrics and alerting
- **Error Handling**: Comprehensive try-catch blocks with user-friendly fallbacks

This architecture represents the **actual production system** deployed and operational for MawinguOps farming advisory platform.