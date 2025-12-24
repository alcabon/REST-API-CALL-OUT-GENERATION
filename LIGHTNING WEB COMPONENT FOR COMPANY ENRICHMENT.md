# **LIGHTNING WEB COMPONENT FOR COMPANY ENRICHMENT**

## **Complete LWC Solution**

### **1. Apex Controller: CompanyEnrichmentController.cls**

```apex
/**
 * @description Apex controller for Company Enrichment LWC
 */
public with sharing class CompanyEnrichmentController {
    
    /**
     * @description Enrich a single account by ID
     * @param accountId Account ID to enrich
     * @return EnrichmentResult wrapper
     */
    @AuraEnabled
    public static EnrichmentResult enrichAccount(Id accountId) {
        EnrichmentResult result = new EnrichmentResult();
        
        try {
            // Get account
            Account acc = [
                SELECT Id, Name, Website, Last_Enriched__c
                FROM Account 
                WHERE Id = :accountId 
                LIMIT 1
            ];
            
            if (String.isBlank(acc.Website)) {
                result.success = false;
                result.message = 'Account must have a Website to enrich.';
                return result;
            }
            
            // Check if recently enriched
            if (acc.Last_Enriched__c != null && acc.Last_Enriched__c > System.now().addHours(-24)) {
                result.success = false;
                result.message = 'Account was enriched within the last 24 hours. Please wait before enriching again.';
                result.lastEnriched = acc.Last_Enriched__c;
                return result;
            }
            
            String domain = extractDomain(acc.Website);
            
            // Call enrichment service
            CompanyEnrichmentWrapper.ApiResponseWrapper response = 
                CompanyEnrichmentService.enrichCompany(domain);
            
            if (response.success && response.data != null) {
                // Map and update account
                Account enrichedAcc = CompanyEnrichmentMapper.mapToAccount(response.data, domain);
                enrichedAcc.Id = accountId;
                update enrichedAcc;
                
                // Map and insert contacts
                List<Contact> contacts = CompanyEnrichmentMapper.mapToContacts(response.data, accountId);
                if (contacts != null && !contacts.isEmpty()) {
                    insert contacts;
                    result.contactsCreated = contacts.size();
                }
                
                result.success = true;
                result.message = 'Account enriched successfully!';
                result.accountData = enrichedAcc;
                result.lastEnriched = System.now();
                
            } else {
                result.success = false;
                result.message = 'Failed to enrich account. Please try again.';
            }
            
        } catch (CompanyEnrichmentException e) {
            result.success = false;
            result.message = 'Error: ' + e.getMessage();
        } catch (Exception e) {
            result.success = false;
            result.message = 'Unexpected error: ' + e.getMessage();
            System.debug('Error in enrichAccount: ' + e.getMessage() + '\n' + e.getStackTraceString());
        }
        
        return result;
    }
    
    /**
     * @description Get account enrichment status
     * @param accountId Account ID
     * @return AccountStatus wrapper
     */
    @AuraEnabled(cacheable=true)
    public static AccountStatus getAccountStatus(Id accountId) {
        AccountStatus status = new AccountStatus();
        
        try {
            Account acc = [
                SELECT Id, Name, Website, Industry, NumberOfEmployees, 
                       AnnualRevenue, Description, Last_Enriched__c
                FROM Account 
                WHERE Id = :accountId 
                LIMIT 1
            ];
            
            status.accountName = acc.Name;
            status.website = acc.Website;
            status.hasWebsite = String.isNotBlank(acc.Website);
            status.lastEnriched = acc.Last_Enriched__c;
            status.isEnriched = acc.Last_Enriched__c != null;
            status.industry = acc.Industry;
            status.employeeCount = acc.NumberOfEmployees;
            status.revenue = acc.AnnualRevenue;
            status.description = acc.Description;
            
            // Calculate enrichment score (0-100)
            Integer score = 0;
            if (String.isNotBlank(acc.Industry)) score += 20;
            if (acc.NumberOfEmployees != null) score += 20;
            if (acc.AnnualRevenue != null) score += 20;
            if (String.isNotBlank(acc.Description)) score += 20;
            if (acc.Last_Enriched__c != null) score += 20;
            status.enrichmentScore = score;
            
            // Check if can enrich
            status.canEnrich = String.isNotBlank(acc.Website) && 
                (acc.Last_Enriched__c == null || acc.Last_Enriched__c < System.now().addHours(-24));
            
            // Get related contacts count
            status.contactCount = [SELECT COUNT() FROM Contact WHERE AccountId = :accountId];
            
        } catch (Exception e) {
            System.debug('Error in getAccountStatus: ' + e.getMessage());
        }
        
        return status;
    }
    
    /**
     * @description Get recent enrichment logs for an account
     * @param accountId Account ID
     * @param limitCount Number of logs to return
     * @return List of API_Call_Log__c
     */
    @AuraEnabled(cacheable=true)
    public static List<API_Call_Log__c> getEnrichmentLogs(Id accountId, Integer limitCount) {
        Account acc = [SELECT Website FROM Account WHERE Id = :accountId LIMIT 1];
        String domain = extractDomain(acc.Website);
        
        return [
            SELECT Id, Called_At__c, Success__c, Status_Code__c, 
                   Error_Message__c, Execution_Time_Ms__c
            FROM API_Call_Log__c
            WHERE Endpoint__c LIKE :('%' + domain + '%')
            ORDER BY Called_At__c DESC
            LIMIT :limitCount
        ];
    }
    
    /**
     * @description Trigger batch enrichment for multiple accounts
     * @param accountIds Set of Account IDs
     * @return Batch job ID
     */
    @AuraEnabled
    public static String enrichMultipleAccounts(List<Id> accountIds) {
        try {
            // Queue the enrichment
            QueueableCompanyEnrichment queueable = new QueueableCompanyEnrichment(
                new Set<Id>(accountIds)
            );
            Id jobId = System.enqueueJob(queueable);
            
            return jobId;
        } catch (Exception e) {
            throw new AuraHandledException('Error starting batch enrichment: ' + e.getMessage());
        }
    }
    
    /**
     * @description Get accounts that need enrichment
     * @param limitCount Number of accounts to return
     * @return List of accounts
     */
    @AuraEnabled(cacheable=true)
    public static List<Account> getAccountsNeedingEnrichment(Integer limitCount) {
        return [
            SELECT Id, Name, Website, Industry, Last_Enriched__c, 
                   NumberOfEmployees, AnnualRevenue
            FROM Account
            WHERE Website != null
            AND (Last_Enriched__c = null OR Last_Enriched__c < LAST_N_DAYS:30)
            ORDER BY LastModifiedDate DESC
            LIMIT :limitCount
        ];
    }
    
    /**
     * @description Get enrichment statistics
     * @return EnrichmentStats wrapper
     */
    @AuraEnabled(cacheable=true)
    public static EnrichmentStats getEnrichmentStats() {
        EnrichmentStats stats = new EnrichmentStats();
        
        // Total accounts with website
        stats.totalAccountsWithWebsite = [
            SELECT COUNT() 
            FROM Account 
            WHERE Website != null
        ];
        
        // Enriched accounts
        stats.enrichedAccounts = [
            SELECT COUNT() 
            FROM Account 
            WHERE Last_Enriched__c != null
        ];
        
        // Accounts needing enrichment
        stats.needsEnrichment = [
            SELECT COUNT() 
            FROM Account 
            WHERE Website != null 
            AND (Last_Enriched__c = null OR Last_Enriched__c < LAST_N_DAYS:30)
        ];
        
        // Recent successful enrichments
        stats.recentSuccessful = [
            SELECT COUNT() 
            FROM API_Call_Log__c 
            WHERE Success__c = true 
            AND Called_At__c = LAST_N_DAYS:7
        ];
        
        // Recent failed enrichments
        stats.recentFailed = [
            SELECT COUNT() 
            FROM API_Call_Log__c 
            WHERE Success__c = false 
            AND Called_At__c = LAST_N_DAYS:7
        ];
        
        return stats;
    }
    
    // Helper method
    private static String extractDomain(String website) {
        if (String.isBlank(website)) return null;
        String domain = website.toLowerCase();
        domain = domain.replace('http://', '').replace('https://', '').replace('www.', '');
        if (domain.contains('/')) {
            domain = domain.substring(0, domain.indexOf('/'));
        }
        return domain;
    }
    
    // Wrapper classes
    public class EnrichmentResult {
        @AuraEnabled public Boolean success { get; set; }
        @AuraEnabled public String message { get; set; }
        @AuraEnabled public Account accountData { get; set; }
        @AuraEnabled public Integer contactsCreated { get; set; }
        @AuraEnabled public DateTime lastEnriched { get; set; }
        
        public EnrichmentResult() {
            this.contactsCreated = 0;
        }
    }
    
    public class AccountStatus {
        @AuraEnabled public String accountName { get; set; }
        @AuraEnabled public String website { get; set; }
        @AuraEnabled public Boolean hasWebsite { get; set; }
        @AuraEnabled public Boolean isEnriched { get; set; }
        @AuraEnabled public Boolean canEnrich { get; set; }
        @AuraEnabled public DateTime lastEnriched { get; set; }
        @AuraEnabled public Integer enrichmentScore { get; set; }
        @AuraEnabled public String industry { get; set; }
        @AuraEnabled public Integer employeeCount { get; set; }
        @AuraEnabled public Decimal revenue { get; set; }
        @AuraEnabled public String description { get; set; }
        @AuraEnabled public Integer contactCount { get; set; }
    }
    
    public class EnrichmentStats {
        @AuraEnabled public Integer totalAccountsWithWebsite { get; set; }
        @AuraEnabled public Integer enrichedAccounts { get; set; }
        @AuraEnabled public Integer needsEnrichment { get; set; }
        @AuraEnabled public Integer recentSuccessful { get; set; }
        @AuraEnabled public Integer recentFailed { get; set; }
    }
}
```

---

### **2. Main LWC Component**

#### **companyEnrichment.html**

```html
<template>
    <lightning-card title="Company Enrichment" icon-name="custom:custom63">
        
        <!-- Account Status Section -->
        <div class="slds-p-around_medium">
            
            <!-- Loading Spinner -->
            <template if:true={isLoading}>
                <lightning-spinner alternative-text="Loading..." size="medium"></lightning-spinner>
            </template>
            
            <!-- Account Information -->
            <template if:false={isLoading}>
                
                <!-- Status Overview -->
                <div class="slds-grid slds-gutters slds-m-bottom_medium">
                    <div class="slds-col slds-size_1-of-2">
                        <lightning-tile label="Account" href={accountUrl}>
                            <p class="slds-text-heading_medium">{accountStatus.accountName}</p>
                            <dl class="slds-list_horizontal slds-wrap">
                                <dt class="slds-item_label slds-text-color_weak">Website:</dt>
                                <dd class="slds-item_detail">{accountStatus.website}</dd>
                            </dl>
                        </lightning-tile>
                    </div>
                    
                    <div class="slds-col slds-size_1-of-2">
                        <lightning-tile label="Enrichment Status">
                            <!-- Enrichment Score -->
                            <div class="score-container">
                                <div class="score-circle" style={scoreStyle}>
                                    <span class="score-text">{accountStatus.enrichmentScore}</span>
                                </div>
                                <p class="slds-text-body_small slds-m-top_small">Data Completeness Score</p>
                            </div>
                            
                            <template if:true={accountStatus.lastEnriched}>
                                <p class="slds-text-body_small slds-m-top_small">
                                    Last enriched: <lightning-formatted-date-time 
                                        value={accountStatus.lastEnriched}
                                        year="numeric" month="short" day="numeric"
                                        hour="2-digit" minute="2-digit">
                                    </lightning-formatted-date-time>
                                </p>
                            </template>
                        </lightning-tile>
                    </div>
                </div>
                
                <!-- Alert Messages -->
                <template if:false={accountStatus.hasWebsite}>
                    <div class="slds-notify slds-notify_alert slds-alert_warning slds-m-bottom_medium">
                        <span class="slds-assistive-text">warning</span>
                        <lightning-icon icon-name="utility:warning" size="x-small" 
                                      variant="warning" class="slds-m-right_x-small"></lightning-icon>
                        <h2>This account does not have a website. Please add a website to enable enrichment.</h2>
                    </div>
                </template>
                
                <template if:true={showSuccessMessage}>
                    <div class="slds-notify slds-notify_alert slds-alert_success slds-m-bottom_medium">
                        <lightning-icon icon-name="utility:success" size="x-small" 
                                      variant="success" class="slds-m-right_x-small"></lightning-icon>
                        <h2>{successMessage}</h2>
                    </div>
                </template>
                
                <template if:true={showErrorMessage}>
                    <div class="slds-notify slds-notify_alert slds-alert_error slds-m-bottom_medium">
                        <lightning-icon icon-name="utility:error" size="x-small" 
                                      variant="error" class="slds-m-right_x-small"></lightning-icon>
                        <h2>{errorMessage}</h2>
                    </div>
                </template>
                
                <!-- Enrichment Action -->
                <div class="slds-m-bottom_medium">
                    <lightning-button 
                        variant="brand"
                        label="Enrich Account Data"
                        icon-name="utility:refresh"
                        onclick={handleEnrich}
                        disabled={enrichButtonDisabled}
                        class="slds-m-right_small">
                    </lightning-button>
                    
                    <lightning-button 
                        variant="neutral"
                        label="View History"
                        icon-name="utility:list"
                        onclick={toggleHistory}
                        class="slds-m-right_small">
                    </lightning-button>
                    
                    <lightning-button 
                        variant="neutral"
                        label="Refresh Status"
                        icon-name="utility:sync"
                        onclick={refreshStatus}>
                    </lightning-button>
                </div>
                
                <!-- Current Data Display -->
                <template if:true={accountStatus.isEnriched}>
                    <div class="slds-box slds-m-bottom_medium">
                        <h3 class="slds-text-heading_small slds-m-bottom_small">Current Enriched Data</h3>
                        
                        <lightning-layout multiple-rows="true">
                            <lightning-layout-item size="6" padding="around-small">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Industry</label>
                                    <div class="slds-form-element__control">
                                        <lightning-formatted-text value={accountStatus.industry}></lightning-formatted-text>
                                    </div>
                                </div>
                            </lightning-layout-item>
                            
                            <lightning-layout-item size="6" padding="around-small">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Employees</label>
                                    <div class="slds-form-element__control">
                                        <lightning-formatted-number value={accountStatus.employeeCount}></lightning-formatted-number>
                                    </div>
                                </div>
                            </lightning-layout-item>
                            
                            <lightning-layout-item size="6" padding="around-small">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Annual Revenue</label>
                                    <div class="slds-form-element__control">
                                        <lightning-formatted-number 
                                            value={accountStatus.revenue} 
                                            format-style="currency"
                                            currency-code="USD">
                                        </lightning-formatted-number>
                                    </div>
                                </div>
                            </lightning-layout-item>
                            
                            <lightning-layout-item size="6" padding="around-small">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Contacts</label>
                                    <div class="slds-form-element__control">
                                        <lightning-formatted-number value={accountStatus.contactCount}></lightning-formatted-number>
                                    </div>
                                </div>
                            </lightning-layout-item>
                            
                            <template if:true={accountStatus.description}>
                                <lightning-layout-item size="12" padding="around-small">
                                    <div class="slds-form-element">
                                        <label class="slds-form-element__label">Description</label>
                                        <div class="slds-form-element__control">
                                            <lightning-formatted-text value={accountStatus.description}></lightning-formatted-text>
                                        </div>
                                    </div>
                                </lightning-layout-item>
                            </template>
                        </lightning-layout>
                    </div>
                </template>
                
                <!-- Enrichment History -->
                <template if:true={showHistory}>
                    <div class="slds-box">
                        <h3 class="slds-text-heading_small slds-m-bottom_small">Enrichment History</h3>
                        
                        <template if:true={enrichmentLogs}>
                            <template if:true={enrichmentLogs.length}>
                                <table class="slds-table slds-table_bordered slds-table_cell-buffer">
                                    <thead>
                                        <tr class="slds-line-height_reset">
                                            <th scope="col">Date</th>
                                            <th scope="col">Status</th>
                                            <th scope="col">Status Code</th>
                                            <th scope="col">Execution Time</th>
                                            <th scope="col">Message</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <template for:each={enrichmentLogs} for:item="log">
                                            <tr key={log.Id}>
                                                <td>
                                                    <lightning-formatted-date-time 
                                                        value={log.Called_At__c}
                                                        year="numeric" month="short" day="numeric"
                                                        hour="2-digit" minute="2-digit">
                                                    </lightning-formatted-date-time>
                                                </td>
                                                <td>
                                                    <template if:true={log.Success__c}>
                                                        <lightning-badge label="Success" class="slds-badge_success"></lightning-badge>
                                                    </template>
                                                    <template if:false={log.Success__c}>
                                                        <lightning-badge label="Failed" class="slds-badge_error"></lightning-badge>
                                                    </template>
                                                </td>
                                                <td>{log.Status_Code__c}</td>
                                                <td>{log.Execution_Time_Ms__c} ms</td>
                                                <td>{log.Error_Message__c}</td>
                                            </tr>
                                        </template>
                                    </tbody>
                                </table>
                            </template>
                            <template if:false={enrichmentLogs.length}>
                                <p class="slds-text-body_small slds-text-color_weak">No enrichment history available.</p>
                            </template>
                        </template>
                    </div>
                </template>
                
            </template>
        </div>
    </lightning-card>
</template>
```

#### **companyEnrichment.js**

```javascript
import { LightningElement, api, wire, track } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { refreshApex } from '@salesforce/apex';
import { NavigationMixin } from 'lightning/navigation';

import getAccountStatus from '@salesforce/apex/CompanyEnrichmentController.getAccountStatus';
import enrichAccount from '@salesforce/apex/CompanyEnrichmentController.enrichAccount';
import getEnrichmentLogs from '@salesforce/apex/CompanyEnrichmentController.getEnrichmentLogs';

export default class CompanyEnrichment extends NavigationMixin(LightningElement) {
    @api recordId; // Account Id from record page
    
    @track accountStatus = {};
    @track enrichmentLogs = [];
    @track isLoading = true;
    @track isEnriching = false;
    @track showHistory = false;
    @track showSuccessMessage = false;
    @track showErrorMessage = false;
    @track successMessage = '';
    @track errorMessage = '';
    
    wiredAccountStatusResult;
    wiredLogsResult;
    
    // Wire to get account status
    @wire(getAccountStatus, { accountId: '$recordId' })
    wiredAccountStatus(result) {
        this.wiredAccountStatusResult = result;
        if (result.data) {
            this.accountStatus = result.data;
            this.isLoading = false;
        } else if (result.error) {
            this.showError('Error loading account status: ' + this.getErrorMessage(result.error));
            this.isLoading = false;
        }
    }
    
    // Wire to get enrichment logs
    @wire(getEnrichmentLogs, { accountId: '$recordId', limitCount: 10 })
    wiredLogs(result) {
        this.wiredLogsResult = result;
        if (result.data) {
            this.enrichmentLogs = result.data;
        } else if (result.error) {
            console.error('Error loading logs:', result.error);
        }
    }
    
    // Computed properties
    get enrichButtonDisabled() {
        return !this.accountStatus.canEnrich || this.isEnriching || this.isLoading;
    }
    
    get accountUrl() {
        return `/${this.recordId}`;
    }
    
    get scoreStyle() {
        const score = this.accountStatus.enrichmentScore || 0;
        let color = '#e74c3c'; // Red
        if (score >= 80) color = '#27ae60'; // Green
        else if (score >= 50) color = '#f39c12'; // Orange
        
        return `background: conic-gradient(${color} ${score * 3.6}deg, #ecf0f1 0deg);`;
    }
    
    // Handle enrichment
    handleEnrich() {
        this.isEnriching = true;
        this.showSuccessMessage = false;
        this.showErrorMessage = false;
        
        enrichAccount({ accountId: this.recordId })
            .then(result => {
                if (result.success) {
                    this.successMessage = result.message;
                    if (result.contactsCreated > 0) {
                        this.successMessage += ` ${result.contactsCreated} contact(s) created.`;
                    }
                    this.showSuccessMessage = true;
                    
                    // Show toast
                    this.showToast('Success', this.successMessage, 'success');
                    
                    // Refresh data
                    this.refreshAllData();
                } else {
                    this.errorMessage = result.message;
                    this.showErrorMessage = true;
                    this.showToast('Error', result.message, 'error');
                }
            })
            .catch(error => {
                this.errorMessage = this.getErrorMessage(error);
                this.showErrorMessage = true;
                this.showToast('Error', this.errorMessage, 'error');
            })
            .finally(() => {
                this.isEnriching = false;
            });
    }
    
    // Toggle history
    toggleHistory() {
        this.showHistory = !this.showHistory;
    }
    
    // Refresh status
    refreshStatus() {
        this.isLoading = true;
        this.refreshAllData();
    }
    
    // Refresh all data
    refreshAllData() {
        Promise.all([
            refreshApex(this.wiredAccountStatusResult),
            refreshApex(this.wiredLogsResult)
        ]).then(() => {
            this.isLoading = false;
        }).catch(error => {
            this.showError('Error refreshing data: ' + this.getErrorMessage(error));
            this.isLoading = false;
        });
    }
    
    // Show toast notification
    showToast(title, message, variant) {
        const evt = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant,
        });
        this.dispatchEvent(evt);
    }
    
    // Show error
    showError(message) {
        this.errorMessage = message;
        this.showErrorMessage = true;
        this.showToast('Error', message, 'error');
    }
    
    // Get error message from error object
    getErrorMessage(error) {
        if (error.body && error.body.message) {
            return error.body.message;
        } else if (error.message) {
            return error.message;
        } else if (typeof error === 'string') {
            return error;
        }
        return 'An unknown error occurred';
    }
}
```

#### **companyEnrichment.css**

```css
.score-container {
    text-align: center;
}

.score-circle {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto;
    position: relative;
}

.score-circle::before {
    content: '';
    position: absolute;
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background: white;
}

.score-text {
    position: relative;
    z-index: 1;
    font-size: 24px;
    font-weight: bold;
    color: #2c3e50;
}

.slds-badge_success {
    background-color: #27ae60;
}

.slds-badge_error {
    background-color: #e74c3c;
}
```

#### **companyEnrichment.js-meta.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__RecordPage</target>
        <target>lightning__AppPage</target>
        <target>lightning__HomePage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <objects>
                <object>Account</object>
            </objects>
        </targetConfig>
    </targetConfigs>
    <masterLabel>Company Enrichment</masterLabel>
    <description>Enriches account data using external API</description>
</LightningComponentBundle>
```

---

### **3. Bulk Enrichment Component**

#### **bulkCompanyEnrichment.html**

```html
<template>
    <lightning-card title="Bulk Company Enrichment" icon-name="custom:custom88">
        <div class="slds-p-around_medium">
            
            <!-- Statistics Dashboard -->
            <template if:true={stats}>
                <div class="slds-grid slds-gutters slds-m-bottom_large">
                    <div class="slds-col slds-size_1-of-5">
                        <article class="slds-tile slds-tile_board stats-tile">
                            <h3 class="slds-truncate" title="Total Accounts">
                                <span class="slds-text-heading_medium">{stats.totalAccountsWithWebsite}</span>
                            </h3>
                            <div class="slds-tile__detail">
                                <p class="slds-text-body_small">Total Accounts</p>
                            </div>
                        </article>
                    </div>
                    
                    <div class="slds-col slds-size_1-of-5">
                        <article class="slds-tile slds-tile_board stats-tile stats-enriched">
                            <h3 class="slds-truncate">
                                <span class="slds-text-heading_medium">{stats.enrichedAccounts}</span>
                            </h3>
                            <div class="slds-tile__detail">
                                <p class="slds-text-body_small">Enriched</p>
                            </div>
                        </article>
                    </div>
                    
                    <div class="slds-col slds-size_1-of-5">
                        <article class="slds-tile slds-tile_board stats-tile stats-needs">
                            <h3 class="slds-truncate">
                                <span class="slds-text-heading_medium">{stats.needsEnrichment}</span>
                            </h3>
                            <div class="slds-tile__detail">
                                <p class="slds-text-body_small">Needs Enrichment</p>
                            </div>
                        </article>
                    </div>
                    
                    <div class="slds-col slds-size_1-of-5">
                        <article class="slds-tile slds-tile_board stats-tile stats-success">
                            <h3 class="slds-truncate">
                                <span class="slds-text-heading_medium">{stats.recentSuccessful}</span>
                            </h3>
                            <div class="slds-tile__detail">
                                <p class="slds-text-body_small">Success (7 days)</p>
                            </div>
                        </article>
                    </div>
                    
                    <div class="slds-col slds-size_1-of-5">
                        <article class="slds-tile slds-tile_board stats-tile stats-failed">
                            <h3 class="slds-truncate">
                                <span class="slds-text-heading_medium">{stats.recentFailed}</span>
                            </h3>
                            <div class="slds-tile__detail">
                                <p class="slds-text-body_small">Failed (7 days)</p>
                            </div>
                        </article>
                    </div>
                </div>
            </template>
            
            <!-- Action Buttons -->
            <div class="slds-m-bottom_medium">
                <lightning-button 
                    variant="brand"
                    label="Enrich Selected Accounts"
                    icon-name="utility:refresh"
                    onclick={handleBulkEnrich}
                    disabled={bulkEnrichDisabled}>
                </lightning-button>
                
                <lightning-button 
                    variant="neutral"
                    label="Refresh List"
                    icon-name="utility:sync"
                    onclick={refreshList}
                    class="slds-m-left_small">
                </lightning-button>
            </div>
            
            <!-- Accounts List -->
            <template if:true={accounts}>
                <lightning-datatable
                    key-field="Id"
                    data={accounts}
                    columns={columns}
                    onrowselection={handleRowSelection}
                    hide-checkbox-column={false}
                    show-row-number-column={true}>
                </lightning-datatable>
                
                <div class="slds-m-top_small slds-text-body_small">
                    <p>{selectedCount} account(s) selected</p>
                </div>
            </template>
            
            <!-- Loading Spinner -->
            <template if:true={isLoading}>
                <lightning-spinner alternative-text="Loading..."></lightning-spinner>
            </template>
            
        </div>
    </lightning-card>
</template>
```

#### **bulkCompanyEnrichment.js**

```javascript
import { LightningElement, track, wire } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { refreshApex } from '@salesforce/apex';

import getAccountsNeedingEnrichment from '@salesforce/apex/CompanyEnrichmentController.getAccountsNeedingEnrichment';
import enrichMultipleAccounts from '@salesforce/apex/CompanyEnrichmentController.enrichMultipleAccounts';
import getEnrichmentStats from '@salesforce/apex/CompanyEnrichmentController.getEnrichmentStats';

const COLUMNS = [
    { label: 'Account Name', fieldName: 'Name', type: 'text' },
    { label: 'Website', fieldName: 'Website', type: 'url' },
    { label: 'Industry', fieldName: 'Industry', type: 'text' },
    { 
        label: 'Employees', 
        fieldName: 'NumberOfEmployees', 
        type: 'number',
        cellAttributes: { alignment: 'left' }
    },
    { 
        label: 'Revenue', 
        fieldName: 'AnnualRevenue', 
        type: 'currency',
        typeAttributes: { currencyCode: 'USD' }
    },
    { 
        label: 'Last Enriched', 
        fieldName: 'Last_Enriched__c', 
        type: 'date',
        typeAttributes: {
            year: 'numeric',
            month: 'short',
            day: 'numeric'
        }
    }
];

export default class BulkCompanyEnrichment extends LightningElement {
    @track accounts = [];
    @track selectedAccounts = [];
    @track stats = {};
    @track isLoading = true;
    
    columns = COLUMNS;
    wiredAccountsResult;
    wiredStatsResult;
    
    @wire(getAccountsNeedingEnrichment, { limitCount: 100 })
    wiredAccounts(result) {
        this.wiredAccountsResult = result;
        if (result.data) {
            this.accounts = result.data;
            this.isLoading = false;
        } else if (result.error) {
            this.showToast('Error', 'Error loading accounts: ' + this.getErrorMessage(result.error), 'error');
            this.isLoading = false;
        }
    }
    
    @wire(getEnrichmentStats)
    wiredStats(result) {
        this.wiredStatsResult = result;
        if (result.data) {
            this.stats = result.data;
        } else if (result.error) {
            console.error('Error loading stats:', result.error);
        }
    }
    
    get selectedCount() {
        return this.selectedAccounts.length;
    }
    
    get bulkEnrichDisabled() {
        return this.selectedAccounts.length === 0 || this.isLoading;
    }
    
    handleRowSelection(event) {
        const selectedRows = event.detail.selectedRows;
        this.selectedAccounts = selectedRows.map(row => row.Id);
    }
    
    handleBulkEnrich() {
        if (this.selectedAccounts.length === 0) {
            this.showToast('Warning', 'Please select at least one account', 'warning');
            return;
        }
        
        this.isLoading = true;
        
        enrichMultipleAccounts({ accountIds: this.selectedAccounts })
            .then(jobId => {
                this.showToast(
                    'Success', 
                    `Bulk enrichment started for ${this.selectedAccounts.length} account(s). Job ID: ${jobId}`,
                    'success'
                );
                
                // Clear selection
                this.selectedAccounts = [];
                
                // Refresh data after a delay
                setTimeout(() => {
                    this.refreshList();
                }, 5000);
            })
            .catch(error => {
                this.showToast('Error', this.getErrorMessage(error), 'error');
            })
            .finally(() => {
                this.isLoading = false;
            });
    }
    
    refreshList() {
        this.isLoading = true;
        Promise.all([
            refreshApex(this.wiredAccountsResult),
            refreshApex(this.wiredStatsResult)
        ]).then(() => {
            this.isLoading = false;
            this.showToast('Success', 'List refreshed', 'success');
        }).catch(error => {
            this.showToast('Error', this.getErrorMessage(error), 'error');
            this.isLoading = false;
        });
    }
    
    showToast(title, message, variant) {
        const evt = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant,
        });
        this.dispatchEvent(evt);
    }
    
    getErrorMessage(error) {
        if (error.body && error.body.message) {
            return error.body.message;
        } else if (error.message) {
            return error.message;
        } else if (typeof error === 'string') {
            return error;
        }
        return 'An unknown error occurred';
    }
}
```

#### **bulkCompanyEnrichment.css**

```css
.stats-tile {
    text-align: center;
    padding: 1rem;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border-radius: 8px;
}

.stats-enriched {
    background: linear-gradient(135deg, #27ae60 0%, #229954 100%);
}

.stats-needs {
    background: linear-gradient(135deg, #f39c12 0%, #e67e22 100%);
}

.stats-success {
    background: linear-gradient(135deg, #3498db 0%, #2980b9 100%);
}

.stats-failed {
    background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
}

.stats-tile h3,
.stats-tile p {
    color: white;
}
```

#### **bulkCompanyEnrichment.js-meta.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__AppPage</target>
        <target>lightning__HomePage</target>
    </targets>
    <masterLabel>Bulk Company Enrichment</masterLabel>
    <description>Bulk enrichment for multiple accounts</description>
</LightningComponentBundle>
```

---

### **4. Test Class for Controller**

#### **CompanyEnrichmentController_Test.cls**

```apex
@isTest
private class CompanyEnrichmentController_Test {
    
    @testSetup
    static void setup() {
        Account acc = new Account(
            Name = 'Test Account',
            Website = 'acme.com'
        );
        insert acc;
        
        // Create contacts
        List<Contact> contacts = new List<Contact>();
        contacts.add(new Contact(
            FirstName = 'John',
            LastName = 'Doe',
            Email = 'john@acme.com',
            AccountId = acc.Id
        ));
        insert contacts;
    }
    
    @isTest
    static void testEnrichAccount_Success() {
        Test.setMock(HttpCalloutMock.class, new CompanyEnrichmentMock(200, true));
        
        Account acc = [SELECT Id FROM Account LIMIT 1];
        
        Test.startTest();
        CompanyEnrichmentController.EnrichmentResult result = 
            CompanyEnrichmentController.enrichAccount(acc.Id);
        Test.stopTest();
        
        System.assertEquals(true, result.success, 'Enrichment should succeed');
        System.assertNotEquals(null, result.accountData, 'Account data should be returned');
    }
    
    @isTest
    static void testEnrichAccount_NoWebsite() {
        Account acc = new Account(Name = 'No Website Account');
        insert acc;
        
        Test.startTest();
        CompanyEnrichmentController.EnrichmentResult result = 
            CompanyEnrichmentController.enrichAccount(acc.Id);
        Test.stopTest();
        
        System.assertEquals(false, result.success, 'Should fail without website');
        System.assert(result.message.contains('Website'), 'Error message should mention website');
    }
    
    @isTest
    static void testGetAccountStatus() {
        Account acc = [SELECT Id FROM Account LIMIT 1];
        
        Test.startTest();
        CompanyEnrichmentController.AccountStatus status = 
            CompanyEnrichmentController.getAccountStatus(acc.Id);
        Test.stopTest();
        
        System.assertEquals('Test Account', status.accountName);
        System.assertEquals(true, status.hasWebsite);
        System.assertEquals(1, status.contactCount);
    }
    
    @isTest
    static void testGetEnrichmentStats() {
        Test.startTest();
        CompanyEnrichmentController.EnrichmentStats stats = 
            CompanyEnrichmentController.getEnrichmentStats();
        Test.stopTest();
        
        System.assertNotEquals(null, stats);
        System.assertEquals(1, stats.totalAccountsWithWebsite);
    }
    
    @isTest
    static void testEnrichMultipleAccounts() {
        Test.setMock(HttpCalloutMock.class, new CompanyEnrichmentMock(200, true));
        
        Account acc = [SELECT Id FROM Account LIMIT 1];
        List<Id> accountIds = new List<Id>{acc.Id};
        
        Test.startTest();
        String jobId = CompanyEnrichmentController.enrichMultipleAccounts(accountIds);
        Test.stopTest();
        
        System.assertNotEquals(null, jobId, 'Job ID should be returned');
    }
}
```

---

## **DEPLOYMENT STEPS**

### **1. Deploy Apex Classes (in order)**
```
1. CompanyEnrichmentException.cls
2. CompanyEnrichmentWrapper.cls
3. APIConfigurationManager.cls
4. CompanyEnrichmentMapper.cls
5. CompanyEnrichmentService.cls
6. QueueableCompanyEnrichment.cls
7. BatchCompanyEnrichment.cls
8. CompanyEnrichmentController.cls
9. CompanyEnrichmentMock.cls
10. All test classes
```

### **2. Deploy LWC Components**
```
1. companyEnrichment
2. bulkCompanyEnrichment
```

### **3. Add Components to Lightning Pages**

**For Account Record Page:**
1. Go to Account record page
2. Edit Page
3. Drag "Company Enrichment" component onto page
4. Save & Activate

**For App/Home Page:**
1. Create new Lightning App Page or edit existing
2. Drag "Bulk Company Enrichment" component
3. Save & Activate

---

## **FEATURES SUMMARY**

### **Single Account Enrichment (companyEnrichment)**
- ✅ Visual enrichment score (0-100)
- ✅ One-click enrichment button
- ✅ Real-time status updates
- ✅ Enrichment history table
- ✅ Current data display
- ✅ Success/error notifications
- ✅ 24-hour cooldown enforcement

### **Bulk Enrichment (bulkCompanyEnrichment)**
- ✅ Statistics dashboard
- ✅ Multi-select datatable
- ✅ Bulk enrichment trigger
- ✅ Progress tracking
- ✅ Real-time metrics

---

Would you like me to add any additional features like:
1. **Scheduled enrichment UI** (schedule batch jobs from UI)
2. **AI-powered recommendations** (suggest which accounts to enrich first)
3. **Export enrichment reports** (download CSV of enrichment results)
4. **Webhook notifications** (Slack/Teams integration)?
