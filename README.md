# Adobe Acrobat Sign for Salesforce: Developer Guide

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









