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

