# **BEST INPUT FORMATS FOR SALESFORCE OBJECT/FIELD DEFINITIONS**

## **RECOMMENDED: YAML FORMAT** ⭐ (Most Comprehensive)

YAML provides the best balance of human readability and structured data for AI consumption.

```yaml
# Salesforce Object Schema Definition
# Version: 1.0
# Last Updated: 2024-12-24

objects:
  
  Account:
    api_name: Account
    label: Account
    plural_label: Accounts
    description: Standard Salesforce Account object with custom fields
    type: standard
    
    custom_fields:
      
      - api_name: YearStarted__c
        label: Year Started
        type: Number
        length: 4
        decimal_places: 0
        required: false
        unique: false
        external_id: false
        default_value: null
        help_text: Year the company was founded
        description: Captures the founding year from API enrichment
        field_level_security:
          read: all_users
          edit: system_admin, api_user
      
      - api_name: LinkedIn__c
        label: LinkedIn Profile
        type: URL
        length: 255
        required: false
        help_text: Company LinkedIn profile URL
        validation_rules:
          - name: Valid_LinkedIn_URL
            formula: "AND(NOT(ISBLANK(LinkedIn__c)), NOT(CONTAINS(LinkedIn__c, 'linkedin.com')))"
            error_message: Must be a valid LinkedIn URL
      
      - api_name: Twitter__c
        label: Twitter Handle
        type: Text
        length: 255
        required: false
        help_text: Company Twitter handle (e.g., @company)
      
      - api_name: Last_Enriched__c
        label: Last Enriched Date
        type: DateTime
        required: false
        description: Timestamp of last successful API enrichment
        indexed: true
      
      - api_name: Enrichment_Status__c
        label: Enrichment Status
        type: Picklist
        required: false
        picklist_values:
          - label: Not Enriched
            value: Not_Enriched
            default: true
          - label: Enriched
            value: Enriched
          - label: Failed
            value: Failed
          - label: In Progress
            value: In_Progress
        restricted: true
        
      - api_name: Data_Source__c
        label: Data Source
        type: Text
        length: 100
        required: false
        description: Source of enrichment data (e.g., CompanyEnrichmentAPI)
  
  Contact:
    api_name: Contact
    label: Contact
    type: standard
    
    relationships:
      - field: AccountId
        type: Lookup
        reference_to: Account
        relationship_name: Account
        required: false
        cascade_delete: false
    
    custom_fields:
      
      - api_name: Source__c
        label: Source
        type: Picklist
        required: false
        picklist_values:
          - label: Manual Entry
            value: Manual
            default: true
          - label: API Enrichment
            value: API
          - label: Data Import
            value: Import
          - label: Web Form
            value: Web
  
  API_Call_Log__c:
    api_name: API_Call_Log__c
    label: API Call Log
    plural_label: API Call Logs
    type: custom
    description: Logs all external API callouts for auditing and debugging
    sharing_model: Private
    enable_reports: true
    enable_activities: false
    enable_history: false
    
    fields:
      
      - api_name: Name
        label: Log Name
        type: AutoNumber
        display_format: "LOG-{0000000}"
        description: Auto-generated log identifier
      
      - api_name: Endpoint__c
        label: Endpoint
        type: Text
        length: 255
        required: true
        help_text: Full API endpoint URL
        indexed: true
      
      - api_name: Request_Body__c
        label: Request Body
        type: LongTextArea
        length: 32768
        required: false
        visible_lines: 10
      
      - api_name: Response_Body__c
        label: Response Body
        type: LongTextArea
        length: 32768
        required: false
        visible_lines: 10
      
      - api_name: Status_Code__c
        label: HTTP Status Code
        type: Number
        length: 3
        decimal_places: 0
        required: false
        help_text: HTTP response status code (200, 404, 500, etc.)
      
      - api_name: Success__c
        label: Success
        type: Checkbox
        default_value: false
        required: false
      
      - api_name: Error_Message__c
        label: Error Message
        type: LongTextArea
        length: 32768
        required: false
      
      - api_name: Execution_Time_Ms__c
        label: Execution Time (ms)
        type: Number
        length: 18
        decimal_places: 0
        required: false
        help_text: Total execution time in milliseconds
      
      - api_name: Called_At__c
        label: Called At
        type: DateTime
        required: true
        default_value: NOW()
        indexed: true
      
      - api_name: Related_Account__c
        label: Related Account
        type: Lookup
        reference_to: Account
        relationship_name: API_Logs
        required: false
    
    validation_rules:
      - name: Success_Requires_200_Status
        formula: "AND(Success__c = true, Status_Code__c <> 200)"
        error_message: Success can only be true when Status Code is 200
        active: true
    
    indexes:
      - fields: [Called_At__c, Success__c]
        name: Performance_Index
      - fields: [Endpoint__c]
        name: Endpoint_Index

  API_Configuration__mdt:
    api_name: API_Configuration__mdt
    label: API Configuration
    type: custom_metadata
    description: Stores API configuration settings
    visibility: Public
    
    fields:
      
      - api_name: API_Key__c
        label: API Key
        type: Text
        length: 255
        required: true
        encrypted: true
        masked: true
      
      - api_name: Endpoint__c
        label: Endpoint URL
        type: URL
        length: 255
        required: true
      
      - api_name: Timeout__c
        label: Timeout (ms)
        type: Number
        length: 5
        decimal_places: 0
        default_value: 30000
        required: true
      
      - api_name: Max_Retries__c
        label: Max Retries
        type: Number
        length: 1
        decimal_places: 0
        default_value: 3
        required: true
      
      - api_name: Rate_Limit_Per_Hour__c
        label: Rate Limit Per Hour
        type: Number
        length: 4
        decimal_places: 0
        default_value: 1000
        required: true
      
      - api_name: Active__c
        label: Active
        type: Checkbox
        default_value: true

# Field Mappings (API to Salesforce)
field_mappings:
  
  account_mapping:
    source: CompanyEnrichmentAPI
    target_object: Account
    mappings:
      
      - source_field: data.name
        target_field: Name
        required: true
        transformation: truncate(255)
        data_type: String
      
      - source_field: data.domain
        target_field: Website
        required: false
        transformation: truncate(255)
        data_type: String
      
      - source_field: data.description
        target_field: Description
        required: false
        transformation: none
        data_type: LongText
      
      - source_field: data.industry
        target_field: Industry
        required: false
        transformation: truncate(40)
        data_type: String
        validation: must_match_picklist
      
      - source_field: data.employeeCount
        target_field: NumberOfEmployees
        required: false
        transformation: none
        data_type: Integer
      
      - source_field: data.revenue
        target_field: AnnualRevenue
        required: false
        transformation: none
        data_type: Currency
      
      - source_field: data.foundedYear
        target_field: YearStarted__c
        required: false
        transformation: none
        data_type: Integer
      
      - source_field: data.address.street
        target_field: BillingStreet
        required: false
        transformation: truncate(255)
        data_type: String
      
      - source_field: data.address.city
        target_field: BillingCity
        required: false
        transformation: truncate(40)
        data_type: String
      
      - source_field: data.address.state
        target_field: BillingState
        required: false
        transformation: truncate(80)
        data_type: String
      
      - source_field: data.address.country
        target_field: BillingCountry
        required: false
        transformation: truncate(80)
        data_type: String
      
      - source_field: data.address.postalCode
        target_field: BillingPostalCode
        required: false
        transformation: truncate(20)
        data_type: String
      
      - source_field: data.socialMedia.linkedin
        target_field: LinkedIn__c
        required: false
        transformation: truncate(255)
        data_type: URL
      
      - source_field: data.socialMedia.twitter
        target_field: Twitter__c
        required: false
        transformation: truncate(255)
        data_type: String
  
  contact_mapping:
    source: CompanyEnrichmentAPI
    target_object: Contact
    mappings:
      
      - source_field: contacts[].firstName
        target_field: FirstName
        required: false
        transformation: truncate(40)
        data_type: String
      
      - source_field: contacts[].lastName
        target_field: LastName
        required: true
        transformation: truncate(80)
        data_type: String
        default_on_null: Unknown
      
      - source_field: contacts[].email
        target_field: Email
        required: false
        transformation: truncate(80)
        data_type: Email
        validation: valid_email_format
      
      - source_field: contacts[].title
        target_field: Title
        required: false
        transformation: truncate(128)
        data_type: String
      
      - source_field: contacts[].phone
        target_field: Phone
        required: false
        transformation: truncate(40)
        data_type: Phone

# Relationships
relationships:
  
  - name: Account_to_Contacts
    parent_object: Account
    child_object: Contact
    type: Lookup
    field: AccountId
    cascade_delete: false
    relationship_field: Contacts
    cardinality: one_to_many
  
  - name: Account_to_API_Logs
    parent_object: Account
    child_object: API_Call_Log__c
    type: Lookup
    field: Related_Account__c
    cascade_delete: false
    relationship_field: API_Logs__r
    cardinality: one_to_many

# Business Rules & Constraints
business_rules:
  
  - rule_name: Prevent_Duplicate_Enrichment
    description: Don't enrich accounts enriched within 24 hours
    objects: [Account]
    logic: Last_Enriched__c must be > 24 hours ago
    enforcement: Application_Logic
  
  - rule_name: Website_Required_For_Enrichment
    description: Account must have Website to be enriched
    objects: [Account]
    logic: Website field must not be blank
    enforcement: Application_Logic
  
  - rule_name: Unique_Contact_Email_Per_Account
    description: Prevent duplicate contact emails per account
    objects: [Contact]
    logic: Email + AccountId combination must be unique
    enforcement: Application_Logic

# Indexes for Performance
indexes:
  
  - object: Account
    fields: [Website, Last_Enriched__c]
    name: Enrichment_Query_Index
    unique: false
  
  - object: API_Call_Log__c
    fields: [Called_At__c, Success__c]
    name: Log_Performance_Index
    unique: false
  
  - object: Contact
    fields: [Email, AccountId]
    name: Contact_Email_Account_Index
    unique: false
```

---

## **ALTERNATIVE: JSON SCHEMA FORMAT** (Most Precise)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Salesforce Object Schema",
  "description": "Complete object and field definitions for Company Enrichment",
  "version": "1.0",
  
  "objects": {
    "Account": {
      "type": "standard",
      "label": "Account",
      "customFields": [
        {
          "apiName": "YearStarted__c",
          "label": "Year Started",
          "dataType": "Number",
          "precision": 4,
          "scale": 0,
          "required": false,
          "unique": false,
          "externalId": false,
          "helpText": "Year the company was founded",
          "fieldLevelSecurity": {
            "read": ["all"],
            "edit": ["SystemAdmin", "APIUser"]
          }
        },
        {
          "apiName": "LinkedIn__c",
          "label": "LinkedIn Profile",
          "dataType": "Url",
          "length": 255,
          "required": false,
          "validationRules": [
            {
              "name": "Valid_LinkedIn_URL",
              "errorFormula": "AND(NOT(ISBLANK(LinkedIn__c)), NOT(CONTAINS(LinkedIn__c, 'linkedin.com')))",
              "errorMessage": "Must be a valid LinkedIn URL"
            }
          ]
        },
        {
          "apiName": "Last_Enriched__c",
          "label": "Last Enriched Date",
          "dataType": "DateTime",
          "required": false,
          "indexed": true
        },
        {
          "apiName": "Enrichment_Status__c",
          "label": "Enrichment Status",
          "dataType": "Picklist",
          "picklistValues": [
            {"label": "Not Enriched", "value": "Not_Enriched", "default": true},
            {"label": "Enriched", "value": "Enriched"},
            {"label": "Failed", "value": "Failed"},
            {"label": "In Progress", "value": "In_Progress"}
          ],
          "restricted": true
        }
      ]
    },
    
    "API_Call_Log__c": {
      "type": "custom",
      "label": "API Call Log",
      "pluralLabel": "API Call Logs",
      "sharingModel": "Private",
      "enableReports": true,
      "fields": [
        {
          "apiName": "Name",
          "label": "Log Name",
          "dataType": "AutoNumber",
          "displayFormat": "LOG-{0000000}"
        },
        {
          "apiName": "Endpoint__c",
          "label": "Endpoint",
          "dataType": "Text",
          "length": 255,
          "required": true,
          "indexed": true
        },
        {
          "apiName": "Status_Code__c",
          "label": "HTTP Status Code",
          "dataType": "Number",
          "precision": 3,
          "scale": 0
        },
        {
          "apiName": "Success__c",
          "label": "Success",
          "dataType": "Checkbox",
          "defaultValue": false
        }
      ]
    }
  },
  
  "fieldMappings": {
    "accountMapping": {
      "source": "CompanyEnrichmentAPI",
      "target": "Account",
      "mappings": [
        {
          "sourcePath": "data.name",
          "targetField": "Name",
          "required": true,
          "transformation": "truncate(255)",
          "dataType": "string"
        },
        {
          "sourcePath": "data.industry",
          "targetField": "Industry",
          "required": false,
          "transformation": "truncate(40)",
          "validation": "mustMatchPicklist"
        }
      ]
    }
  },
  
  "relationships": [
    {
      "name": "Account_to_Contacts",
      "parentObject": "Account",
      "childObject": "Contact",
      "type": "Lookup",
      "field": "AccountId",
      "cascadeDelete": false
    }
  ]
}
```

---

## **ALTERNATIVE: MARKDOWN TABLE FORMAT** (Most Readable)

```markdown
# Salesforce Object Schema Definition

## Account Object (Standard)

### Custom Fields

| API Name | Label | Type | Length | Required | Default | Help Text | Constraints |
|----------|-------|------|--------|----------|---------|-----------|-------------|
| YearStarted__c | Year Started | Number | 4,0 | No | null | Year company was founded | Range: 1800-2100 |
| LinkedIn__c | LinkedIn Profile | URL | 255 | No | null | Company LinkedIn URL | Must contain 'linkedin.com' |
| Twitter__c | Twitter Handle | Text | 255 | No | null | Company Twitter handle | Format: @username |
| Last_Enriched__c | Last Enriched Date | DateTime | - | No | null | Last successful enrichment | Indexed |
| Enrichment_Status__c | Enrichment Status | Picklist | - | No | Not_Enriched | Current status | Values: Not_Enriched, Enriched, Failed, In_Progress |

### Picklist Values: Enrichment_Status__c

| Value | Label | Default |
|-------|-------|---------|
| Not_Enriched | Not Enriched | Yes |
| Enriched | Enriched | No |
| Failed | Failed | No |
| In_Progress | In Progress | No |

---

## API_Call_Log__c Object (Custom)

**Object Settings:**
- Sharing Model: Private
- Enable Reports: Yes
- Enable Activities: No

### Fields

| API Name | Label | Type | Length | Required | Indexed | Description |
|----------|-------|------|--------|----------|---------|-------------|
| Name | Log Name | AutoNumber | LOG-{0000000} | Yes | Yes | Auto-generated ID |
| Endpoint__c | Endpoint | Text | 255 | Yes | Yes | Full API endpoint URL |
| Request_Body__c | Request Body | LongTextArea | 32768 | No | No | Request payload |
| Response_Body__c | Response Body | LongTextArea | 32768 | No | No | Response payload |
| Status_Code__c | HTTP Status Code | Number | 3,0 | No | No | HTTP status (200, 404, etc.) |
| Success__c | Success | Checkbox | - | No | No | Callout succeeded |
| Error_Message__c | Error Message | LongTextArea | 32768 | No | No | Error details |
| Execution_Time_Ms__c | Execution Time (ms) | Number | 18,0 | No | No | Duration in milliseconds |
| Called_At__c | Called At | DateTime | - | Yes | Yes | Timestamp of callout |
| Related_Account__c | Related Account | Lookup(Account) | - | No | No | Related account record |

---

## Field Mappings: API → Salesforce

### Account Mapping

| API Field Path | Salesforce Field | Required | Transformation | Data Type | Validation |
|----------------|------------------|----------|----------------|-----------|------------|
| data.name | Account.Name | Yes | truncate(255) | String | Non-blank |
| data.domain | Account.Website | No | truncate(255) | String | Valid URL |
| data.description | Account.Description | No | none | LongText | - |
| data.industry | Account.Industry | No | truncate(40) | String | Match picklist |
| data.employeeCount | Account.NumberOfEmployees | No | none | Integer | >= 0 |
| data.revenue | Account.AnnualRevenue | No | none | Currency | >= 0 |
| data.foundedYear | Account.YearStarted__c | No | none | Integer | 1800-2100 |
| data.address.street | Account.BillingStreet | No | truncate(255) | String | - |
| data.address.city | Account.BillingCity | No | truncate(40) | String | - |
| data.socialMedia.linkedin | Account.LinkedIn__c | No | truncate(255) | URL | Valid URL |

### Contact Mapping (Array)

| API Field Path | Salesforce Field | Required | Transformation | Default on Null |
|----------------|------------------|----------|----------------|-----------------|
| contacts[].firstName | Contact.FirstName | No | truncate(40) | - |
| contacts[].lastName | Contact.LastName | Yes | truncate(80) | "Unknown" |
| contacts[].email | Contact.Email | No | truncate(80) | - |
| contacts[].title | Contact.Title | No | truncate(128) | - |
| contacts[].phone | Contact.Phone | No | truncate(40) | - |

---

## Relationships

| Name | Parent | Child | Type | Field | Cascade Delete | Cardinality |
|------|--------|-------|------|-------|----------------|-------------|
| Account_to_Contacts | Account | Contact | Lookup | AccountId | No | 1:N |
| Account_to_API_Logs | Account | API_Call_Log__c | Lookup | Related_Account__c | No | 1:N |

---

## Business Rules & Constraints

### Validation Rules

| Rule Name | Object | Formula | Error Message | Active |
|-----------|--------|---------|---------------|--------|
| Valid_LinkedIn_URL | Account | `AND(NOT(ISBLANK(LinkedIn__c)), NOT(CONTAINS(LinkedIn__c, 'linkedin.com')))` | Must be valid LinkedIn URL | Yes |
| Success_Requires_200_Status | API_Call_Log__c | `AND(Success__c = true, Status_Code__c <> 200)` | Success requires 200 status | Yes |

### Application Logic Rules

| Rule Name | Description | Enforcement |
|-----------|-------------|-------------|
| Prevent_Duplicate_Enrichment | Don't enrich within 24 hours | Apex Logic |
| Website_Required_For_Enrichment | Website must exist | Apex Logic |
| Unique_Contact_Email_Per_Account | Email+Account must be unique | Apex Logic |

---

## Indexes

| Object | Fields | Name | Unique |
|--------|--------|------|--------|
| Account | Website, Last_Enriched__c | Enrichment_Query_Index | No |
| API_Call_Log__c | Called_At__c, Success__c | Log_Performance_Index | No |
| Contact | Email, AccountId | Contact_Email_Account_Index | No |
```

---

## **COMPARISON MATRIX**

| Format | Readability | Parsability | Completeness | Versioning | Best For |
|--------|-------------|-------------|--------------|------------|----------|
| **YAML** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | **Most scenarios** |
| JSON Schema | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Strict validation |
| Markdown Tables | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | Documentation |
| Salesforce XML | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Direct deployment |

---

## **RECOMMENDATION FOR AI PROMPT**

**Use YAML format** because:

1. ✅ **Hierarchical structure** - naturally represents object relationships
2. ✅ **Comments supported** - can add context inline
3. ✅ **Concise syntax** - less verbose than JSON or XML
4. ✅ **Human-readable** - easy to review and edit
5. ✅ **LLM-friendly** - models parse it accurately
6. ✅ **Industry standard** - used by Kubernetes, Docker, GitHub Actions

---

## **OPTIMAL PROMPT STRUCTURE**

```
Generate Apex code for [FEATURE] using the following object schema:

<paste YAML schema here>

Additional Requirements:
- Bulkified operations (handle 200 records)
- Error handling with custom exceptions
- Test coverage >85%
- Follow Salesforce best practices
```

This approach gives the AI **complete context** in a **structured, parseable format** that leads to **accurate, production-ready code generation**.
