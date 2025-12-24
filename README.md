# REST-API-CALL-OUT-GENERATION

As an Apex REST API expert, here's an optimal prompt structure for generating comprehensive Apex callout code with AI:

---

## **OPTIMAL PROMPT TEMPLATE**

```
You are an expert Salesforce Apex developer specializing in REST API integrations. 
Generate production-ready Apex code for consuming an external REST API.

## INPUT SPECIFICATIONS

**API Documentation:**
[Paste OpenAPI/Swagger JSON or YAML here]

**Salesforce Objects & Field Mapping:**
- Target Object: [e.g., Account, Custom__c]
- Field Mappings:
  - API_Response_Field → Salesforce_Field__c
  - [Example: "companyName" → Account.Name]
  - [Example: "address.city" → Account.BillingCity]

**Authentication:**
- Type: [OAuth 2.0 / API Key / Basic Auth / Named Credentials]
- Credentials storage: [Named Credential name or Custom Metadata]

## REQUIREMENTS

Generate the following Apex components:

### 1. **Service Class** (API Callout Logic)
- HttpRequest configuration with proper headers
- Dynamic endpoint construction
- Request body serialization
- Response parsing and deserialization
- Comprehensive error handling with custom exceptions
- Retry logic for transient failures
- Logging mechanism (Platform Events or Custom Objects)

### 2. **Wrapper Classes**
- Request payload wrapper (inner classes for nested JSON)
- Response payload wrapper matching API structure
- DTO (Data Transfer Object) for clean mapping

### 3. **Mapper Class**
- Transform API response to Salesforce SObject
- Handle null values and data type conversions
- Support for nested objects and collections
- Field-level error handling

### 4. **Configuration Handler**
- Read endpoints/credentials from Custom Metadata
- Environment-specific configuration (Dev/QA/Prod)
- Rate limiting awareness

### 5. **Test Class** (>85% coverage)
- Mock HttpCalloutMock implementation
- Test positive scenarios
- Test error scenarios (400, 401, 404, 500, timeout)
- Test field mapping accuracy
- Test bulk operations if applicable

## CODE QUALITY STANDARDS

- Follow Salesforce best practices (bulkification, governor limits)
- Use `@future(callout=true)` or Queueable where appropriate
- Implement proper exception handling
- Include inline comments for complex logic
- Use meaningful variable names
- Maximum method complexity: <15
- DML operations outside loops
- SOQL queries outside loops

## EXAMPLE OUTPUT STRUCTURE

```apex
// 1. API Service
public class ExternalAPIService {
    public static ResponseWrapper callEndpoint(RequestWrapper req) {...}
}

// 2. Wrappers
public class RequestWrapper {...}
public class ResponseWrapper {...}

// 3. Mapper
public class APIToSalesforceMapper {
    public static Account mapToAccount(ResponseWrapper resp) {...}
}

// 4. Test Class
@isTest
public class ExternalAPIService_Test {...}
```

## ADDITIONAL CONSIDERATIONS

- Handle pagination if API supports it
- Implement caching strategy if appropriate
- Include debug logs for troubleshooting
- Document governor limit consumption
- Add @AuraEnabled if called from LWC/Aura components
```

---

## **ENHANCED VERSION WITH SPECIFIC EXAMPLE**

If you have a specific API, use this format:

```
Generate Apex code for integrating with the [API Name] REST API.

**SWAGGER SPEC:**
```json
{
  "openapi": "3.0.0",
  "paths": {
    "/api/v1/companies/{id}": {
      "get": {
        "parameters": [...],
        "responses": {...}
      }
    }
  }
}
```

**MAPPING REQUIREMENTS:**
- API → Salesforce
- response.company.name → Account.Name
- response.company.industry → Account.Industry
- response.company.employees → Account.NumberOfEmployees
- response.contacts[] → Contact records (related to Account)

**AUTHENTICATION:**
- OAuth 2.0 via Named Credential: "CompanyAPI_NC"
- Token refresh handled by Salesforce

**SPECIAL REQUIREMENTS:**
- Schedule daily sync using Schedulable
- Log all API calls to API_Log__c custom object
- Send email notification on repeated failures
- Support both single and bulk operations
```

---

## **KEY IMPROVEMENTS TO REQUEST**

1. **Be specific about error scenarios** you need handled
2. **Specify governor limit considerations** (e.g., "Will process 200 records per transaction")
3. **Include sample JSON** from actual API responses
4. **Define retry strategy** (e.g., "Retry 3 times with exponential backoff")
5. **Mention integration patterns** (Real-time vs Batch vs Scheduled)

Would you like me to create a complete example with a specific API (like a public REST API) to demonstrate the generated output?
