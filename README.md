# Adobe Acrobat Sign for Salesforce: Development Guide

**Overview**

Adobe Acrobat Sign for Salesforce: Developer Guide is designed to help Salesforce developers learn about the objects and parameters that are required to integrate your Salesforce package with Adobe Acrobat Sign.

**Integration Guidelines**

* If you need to know when the agreement is fully signed, implement an Apex trigger on the echosign_dev1__SIGN_Agreement__c object, after or before update (depending on the use case and requirements). When the echosign_dev1__Status__c field changes to Signed or Approved or other final statuses, the agreement is completed. 

* If you need to know when each individual signed PDF is inserted, if for example you need to get each intermediate signed PDF, then implement an Apex trigger on the Attachment or ContentVersion objects, after insert, and watch for a parent agreement and a name which ends in "- signed.pdf" or "- approved.pdf" or other final status

* If you need to know when an individual recipient has signed or approved, implement an Apex trigger on the echosign_dev1__SIGN_Recipients__c object, after or before update (depending on the use case and requirements). When the echosign_dev1__Status__c field changes Signed or Approved or other final statuses, the recipient is completed.  

* If you need to know when a particular event which is part of the signing process occurs, such as an agreement being sent for signature or a reminder being sent, a trigger can be created on agreement events object (echosign_dev1__SIGN_AgreementEvent__c) and check for the type of the event

* The Final Agreement Status names for a completed agreement are: "Signed", "Approved", "Accepted", "Form-Filled", and "Delivered"

* The Final Agreement Status names for a terminated agreement are: "Cancelled / Declined", "Canceled / Declined", "Expired"

**Update Order**
* In v21 the order of updates has changed. Below is the sequence in which the agreement and its related objects are updated:

* Attachments
* Recipients
* Agreement (status and its other attributes)
* Agreement Events
* Chatter Feeds 

**Apex Services - Apex Method in use**

* Starting from Acrobat Sign for Salesforce V 21.0, all asynchronous processes (which include automatic updates and data mappings) have been switched from future methods to queueable, an approach recommended by Salesforce.

* With this change, all customizations in the subscriber organization that add jobs to the Salesforce queue as part of the auto-update or data mapping process will fail with an error "System.LimitException: Too many queueable jobs added to the queue: 2". 

* The failure happens because a queueable process can only add one child queueable job, which is already taken up by Acrobat Sign. For details, refer to Queueable Apex Limits.

* Error: "When chaining jobs, you can add only one job from an executing job with System.enqueueJob, which means that only one child job can exist for each parent queueable job. Starting multiple child jobs from the same queueable job isn’t supported."

* The above-stated error occurs when the agreement status does not change or the data mapping does not run correctly. To resolve this error, look for the offending trigger, process builder, or workflow and deactivate it or switch it to use a synchronous call or schedule it for later.

**Agreement Template Service**
  
* The agreement template service is exposed as a global Apex service by the managed package. This allows Apex code outside of the managed package to load agreements based on existing agreement templates. The class and all exposed methods are marked as global to allow such access.

* The Apex service is exposed through the following invocation class: echosign_dev1.AgreementTemplateService
  
  **METHODS**
![image](https://github.com/BalajiMunirathinam/SignSalesforceRepo/assets/43603803/edf66b65-146b-4b54-9d84-a9df373f74ab)
![image](https://github.com/BalajiMunirathinam/SignSalesforceRepo/assets/43603803/578edd77-c2cc-4318-8699-953a1ea3e6de)

**RUNTIME VARIABLES**

The **echosign_dev1.AgreementTemplateVariable** global class has two global fields.

**name :** The variable name, which must match a runtime variable name configured in the agreement template.

**value :** The value of that variable which will be used during the template load. The value depends on where the variable was used. For example, for a recipient it has to be a contact, lead, or user record ID or an email. For a document variable, it must be an attachment record ID.

**RESULT**
Every method either return the ID of the newly created agreement record or throws an exception with a detailed error message if something went wrong during the load operation.

**API Service**
* The Adobe e-Sign API template service is exposed as a global Apex service by the managed package. This allows Apex code outside of the managed package to invoke a set of Adobe e-Sign API's through these wrappers. The wrappers greatly simplify the API invocation because consumers do not need to create request and response data model.
  
* Also consumers do not need to handle the transformation of Salesforce data into e-Sign data models. Most of the complexity is abstracted from the consumer.

* For example, to send an agreement the consumer just passes in the agreement record ID, the service will handle querying it, extracting all of the relevant data, passing it on the API and parsing the result.
  
* The class and all exposed methods are marked as global to allow such access.
  * v17 and below invokes SOAP API's
  * v18 and above invokes REST API's

* The Apex service is exposed through the following invocation class: echosign_dev1.EchoSignApiService
  
  **METHODS**
  ![image](https://github.com/BalajiMunirathinam/SignSalesforceRepo/assets/43603803/03f1333b-f0b3-4ebd-8de1-7470e2493cae)
  ![image](https://github.com/BalajiMunirathinam/SignSalesforceRepo/assets/43603803/09adf22c-f066-4a4f-ab3a-a25849a4bbfc)

**Apex Batch Services**

* Exposes the main e-Sign agreement actions on a bulk level, allowing an operation to be performed on a set of agreements. This class implements the Salesforce Database.Batchable interface. It can process any number of records, which will be broken down into sets of 5 and processing each set as an individual transaction, which allows governor limits to be respected.

* The Apex batch service is exposed through the following invocation class: **echosign_dev1.EchoSignActionBatch**

**PARAMETERS**
* The following parameters must be specified to initialize a batch operation. A list of the agreement record ID's on which to perform the provided action.
  The action to perform, one of the following supported values:

  * Remind
  * Send
  * Cancel
  * Delete
  * Update

    * Current user session ID. Only required for an update action type.
    * Submitter user record, used to notify this user through an email once the bulk processing completes.

**USAGE EXAMPLE**
  **User submitterUser = UserInfo.getUserId();**

  **EchoSignActionBatch batch = new EchoSignActionBatch( agreementIds, 'Remind', UserInfo.getSessionId(), submitterUser); syncProcessId = Database.executeBatch(batch, 5);**

**Agreement Template Batch**
  * Takes in a SOQL query and an agreement template record ID. The query is executed to get a set of master object records, each of which is then run through the provided agreement template to generate an agreement record.
    
  * This class implements the Salesforce Database.Batchable interface. It can process any number of records, which will be broken down into sets of 5 and processing each set as an individual transaction, which allows governor limits to be respected.

  * The record types returned by the SOQL query must match the provided agreement template master object type. For each record, the agreement template service is invoked.
    
  * The Apex batch service is exposed through the following invocation class:
      **echosign_dev1.AgreementTemplateBatch**

**PARAMETERS**
        * The following parameters must be specified to initialize a batch operation.
        
        * SOQL query to execute, must contain the record ID as a selected field. Any other field is optional.
        
        * Agreement template record ID, which will be used in conjunction with the master record ID to load an agreement.

**USAGE EXAMPLE**

    * String agreementTemplateId = [SELECT Id from echosign_dev1__Agreement_Template__c where Name = 'Default Template'];
    
    * String soqlQuery = 'SELECT Id from Contact where Account.IsActive = true';
    
    * AgreementTemplateBatch batch = new AgreementTemplateBatch(soqlQuery, agreementTemplateId); syncProcessId = Database.executeBatch(batch, 5);

**Agreement Template Service Batch**

  *Takes in a list of master object record ID's and the master object type, which are then queried, and each of which is then run through the provided agreement template       to generate an agreement record. This class implements the Salesforce Database.Batchable interface.
  * It can process any number of records, which will be broken down into sets of 5 and processing each set as an individual transaction, which allows governor limits to        be respected.
  * * The master object type provided must match the provided agreement template master object type. For each record, the agreement template service is invoked. 
    * The Apex batch service is exposed through the following invocation class: echosign_dev1.AgreementTemplateServiceBatch

**PARAMETERS**
  * The following parameters must be specified to initialize a batch operation.
    
  * List of master record ID's.
  * Agreement template record ID, which will be used in conjunction with the master records to load an agreement.
  * Master object name to query for the master records.

**USAGE EXAMPLE**

    * String agreementTemplateId = [SELECT Id from echosign_dev1__Agreement_Template__c where Name = 'Default Template'];
       
    * AgreementTemplateBatch batch = new AgreementTemplateServiceBatch(new List<Id>{'01p50000000HoMB'}, agreementTemplateId, 'Contact');
    
    * syncProcessId = Database.executeBatch(batch, 5);

**REST Services**
  * Agreement Template Service
      * The agreement template service is exposed as a Salesforce REST web service by the managed package. This allows external systems outside of the Salesforce org to
          load agreements based on existing agreement templates.
      * Please refer to the Creating REST APIs using Apex REST article for more details on how to access and invoke custom REST Apex services from within Salesforce.
          Invocations must provide a valid session ID for authentication and authorization.

  * The web service is exposed from the following URL:
      * https://<instance_name>.salesforce.com/services/apexrest/echosign_dev1/template/load/<template_id>?masterId=<master_id>&varName1=var Value1&varName2=varValue2
   
  <img width="455" alt="image" src="https://github.com/BalajiMunirathinam/SignSalesforceRepo/assets/43603803/fc66608a-982a-4468-82b4-f5e6f81979b0">

**TEMPLATE ID**
  * The last part of the URL is the ID of the agreement template record in the current Salesforce organization which should be used to load the agreement.
    
  * This part of the URL is optional. If omitted, the agreement template marked as the default will be loaded. If the template ID is omitted and no default agreement
    template ID exists, an error will be returned.

  * The template ID can be in the 15 or 18 character format.

**MASTER ID**
The masterId parameter specifies which master record should be used to load the agreement from the specific agreement template. This parameter is optional, but must be specified for any agreement template which specifies a master object type and references that master object in the template.

The master ID can be in the 15 or 18 character format.

**RUNTIME VARIABLES**
Any additional parameters are used as runtime variables, as name-value pairs, used to populate any runtime variables specified in the agreement template.

**RESULT**
  The REST web service returns a LoadResult object which contains the following fields:
      agreementId : If the agreement load operation was successful, this contains the ID of the newly created agreement record.
      error : If there was any error during the loading of the agreement, this field will contain a detailed error message.

**Background Service**

    * The background service capability allows package consumers to invoke various actions on an agreement object by updating the Background Action (echosign_dev1 
          Background_Actions c) field to the corresponding value. 
    * Once the field value is changed from a blank value or another value to one of the following values, the action is kicked off from a trigger that is part of the 
          e-Sign managed package.

        * Remind
        * Send
        * Cancel
        * Delete
        * Update
        
    * All of the actions execute in an asynchronous future mode, so the status will be stored in the Error field on the agreement.










