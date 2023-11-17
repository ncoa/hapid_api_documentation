# HAPI-D API Documentation
Hi! Welcome to the HAPI-D API documentation. This documentation is meant for external developers that will interact with HAPI-D and will provide you with everything you need to know in order to create great applications.

## Table of Contents
 1. [The Data Model](#data-model)
 2. [Getting Authorized](#authorization)
 3. [Workshops](#workshops)
 
     3.1. [Creating a Workshop](#creating-a-workshop)
 4. [Participants](#participants)
    
     4.1 [Creating a Participant](#creating-a-participant)
 6. [Appendix](#appendix)

## Data Model
Here is a model that provides a high-level overview of objects and their relationships.

![Approved Data Model - DM v 3 Hierachy Model](https://user-images.githubusercontent.com/28845647/271027483-f1433bc9-b910-45e2-8a95-36b3b0489a92.png)


## Authorization
Here we are going to show you how to utilize the 3 legged authorization process.
### Overview

The three-legged OAuth process (3LO) in HAPI-D is used to grant an application the necessary permissions to access data on behalf of the user without revealing the user's credentials to the application. The three "legs" represent the three parties involved in the process: the user, the application, and the HAPI-D platform. This process makes use of `authenticate_code` to ensure secure access to the user's data.

You will need to obtain a `<client_id>` and '<client_secret>' by emailing [hapidhelp@ncoa.org](mailto:hapidhelp@ncoa.org) to authenticate your application with HAPI-D.

This document summarizes the three-legged OAuth process for applications interested in API integration with HAPI-D. For complete documentation of the Oauth 2.0 implementation, please review the documentation found on the [Salesforce Help Site](https://help.salesforce.com/s/articleView?language=en_US&id=sf.remoteaccess_oauth_web_server_flow.htm&type=5)

### Process

The 3LO process consists of the following steps:

1.  **User Authorization:** The user authenticates with HAPI-D and grants the application permission to access their data.
2.  **Application Gets Authorization Code:** HAPI-D redirects the user back to the application, including an authorization code in the URL.
3.  **Application Exchanges Authorization Code for an Access Token:** The application sends the authorization code back to HAPI-D to be exchanged for an access token.

### Step-by-Step Guide

#### Step 1: User Authorization

Your application should redirect the user to HAPI-D authorization endpoint. The URL should be of the following format:

-   **Production HAPI-D:** 
	- `https://ncoa1.my.site.com/services/oauth2/authorize?`
-   **Sandbox HAPI-D:** 
	- `https://ncoa1--<environment>.sandbox.my.site.com/services/oauth2/authorize?`

And is required to have the following URL parameters:

```
response_type=code&
client_id=<client_id>&
redirect_uri=<redirect_uri>
```

The optional URL parameter below is **recommended** and used to implement [PKCE](https://datatracker.ietf.org/doc/rfc7636/).
```
code_challenge=<SHA256 hash value of the code_verifier>
```
-   `<client_id>`: Your application's client ID.
-   `<redirect_uri>`: The URI where Salesforce should redirect the user after authorization.
-   `<SHA256 hash value of the code_verifier>`: Specifies the SHA256 hash value of the code_verifier value in the token request. Set this parameter to help prevent authorization code interception attacks. The value must be base64url-encoded.

#### Step 2: Get Authorization Code

After users approve access to the app, HAPI-D redirects users to the callback URL (`<redirect_uri>`), where they can view the callback with an authorization code.

The first part of the callback is the connected app's callback URL: `<redirect_uri>`. The second part is the authorization code that the connected app uses to get an access token: `code=<authorization code>`. The authorization code expires after 15 minutes.

#### Step 3: Exchange Authorization Code for Access Token

The application can now exchange the authorization code for an access token by sending a POST request to the HAPI-D token request endpoint.

-   **Production HAPI-D:** 
	- `https://ncoa1.my.site.com/services/oauth2/token`
-   **Sandbox HAPI-D:** 
	- `https://ncoa1--<environment>.sandbox.my.site.com/services/oauth2/token`

The request should include the following URL parameters:

-   `grant_type`: Set this to `authorization_code`.
-   `client_id`: Your application's client ID.
-   `client_secret`: Your application's client secret.
-   `code`: The authorization code you received in step 2.
-   `redirect_uri`: The same redirect URI used during user authorization.
-   `code_verifier`: Required only if a code_challenge parameter was specified in the authorization request. Specifies 128 bytes of random data with high entropy to make guessing the code value difficult. Set this parameter to help prevent authorization code interception attacks. The value must be base64url-encoded.

If the request is successful, the response will be a JSON object containing an `access_token` and a `refresh_token`.

### Using the Access Token

Once the application has the access token, it can use it to authenticate requests to Salesforce's API.

To use the access token, include it in the `Authorization` header of the HTTP request. The header should look like:

```
Authorization: Bearer <access_token>
```

For example, to use the access token to query data, you could make a GET request to the `query` endpoint like so:

```
GET /services/data/v50.0/query?q=SELECT+name,id+FROM+Account
Host: ncoa1.my.site.com
Authorization: Bearer <access_token>
```

To interact with the sObject API, you could make a GET request to the `sobjects` endpoint like so:

```
GET /services/data/v50.0/sobjects/Account/<id>
Host: ncoa1.my.site.com
Authorization: Bearer <access_token>
```

Remember to replace `<access_token>` with your actual access token in each of the above examples. If using a sandbox, remember to use the correct host for your environment in the format `ncoa1--<environment>.sandbox.my.site.com`

### Using the Refresh Token

Your app can use the refresh token to get a new access token by sending the following refresh token POST requests to the HAPI-D token endpoint: `/services/oauth2/token`.

```
POST /services/oauth2/token HTTP/1.1
Host: ncoa1.my.site.com
grant_type=refresh_token&
client_id=<client_id>&client_secret=<client_secret>&
refresh_token=<refresh_token>
```

If using a sandbox, remember to use the correct host for your environment in the format `ncoa1--<environment>.sandbox.my.site.com`

If the request is successful, the response will be a JSON object containing an `access_token`.

### Conclusion

Three-legged OAuth provides a secure and user-friendly way for your application to access a user's data on HAPI-D. By following the process outlined in this document, your application can get the necessary permissions to access and manipulate a user's HAPI-D data without ever needing to handle their HAPI-D credentials.

## Workshops
A Workshop is the key object that links Programs, Program Targets, Grantee, Host Organization, Implementation Sites, and Facilitator with Participant pre and post survey data. It will also hold key aggregate information from the Participants related to the Workshop. 

![Screenshot 2022-08-25 222133](https://user-images.githubusercontent.com/28845647/271027226-42c184a4-517c-479b-bb81-18f4a8124eeb.png)

### Creating a Workshop
This is a step-by-step guide to creating a new Workshop.
#### Prerequisites
In order to create a Workshop, you must first find or create the following.
 
 1. Check if the host organization already exists in the HAPI database. Steps are given: [Host Organization look up](#find-or-create-host-organization-account).
 2. If the host organization does not exist, you will be able to add the same. Steps are given: [Adding a Host organization](#find-or-create-host-organization-account).
 3. Check if the Implementation site already exists in HAPI-D: Steps are given: [Implementation Sites look up](#find-or-create-implementation-site). 
 4. If the Implementation Site does not exist, you will be able to add the same. Steps are given: [Adding an Implementation Site](#find-or-create-implementation-site).
 5. Check if the Program already exists in the HAPI database. Steps are given: [Program look up](#find-program)
 6. Retrieve all Funding sources and program targets from the HAPI database and select one from the list. Steps are given: [Program Targets look up](#find-program-target)
 7. Filter the Survey name based on survey ID that was selected from Step 9. Steps are given: [Survey Template lookup](#find-survey-template)
 8. Check if a facilitator already exists in the HAPI database. Steps are given: [Facilitator look up](#find-facilitators). 

#### Find or Create Host Organization Account
Use this API call to query the list of Host Organizations. From there, filter the results based on the Name or Id of your Organization. 
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select Name,+BillingStateCode,+BillingCity,+Host_Organization__c+from+Account+where+Host_Organization__c+in+('Validated','Unvalidated')
~~~
The response gives the Account Name, Billing state code, City. If the Host organization is listed in the response, please select it and keep the ID ready for to create the workshop or else, you can create one.

If your Organization is not found, you can add it using the following API endpoint and request body with your organization's information. You can find allowable values for `Site_Type__c` [here](#picklist-values).
~~~
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/Account
~~~
~~~
{
  "Name": "Your Account Name",
  "Site_Type__c": "Your Site Type",
  "BillingCity": "Your City",
  "BillingStreet": "Your Street Address",
  "BillingState": "Your State",
  "BillingPostalCode": "Your Postal Code"
}
~~~

If the Host Organization was successfully created, an `Id` will be generated and the response will look similar to this:
```
{
    "id": "001DG00001d2li0YER",
    "success": true,
    "errors": []
}
```

#### Find or Create Implementation Site
Use this API call to query the list of Implementation Sites to find yours:
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=Select+id+,+City__c+,+Street__c+,+state__c+,+Name+FROM+epd_implementation_site__c+WHERE+Id+=+implementationSiteId
~~~
The response gives the Implementation Site Name, ID, City, Street and State. If the implementation site is listed in the response, please select the respective ID for workshop creation or else, you can create one.

If your Implementation Site is not listed, use the following API endpoint and request body to create one with your information. You can find allowable values for `Site_Type__c` [here](#picklist-values).
~~~
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_Implementation_Site__c
~~~
~~~
{
  "Name": "Your Implementation Site Name",
  "Site_Type__c": "Your Site Type",
  "City__c": "Your City",
  "Street__c": "Your Street Address",
  "State__c": "Your State",
  "Zipcode__c": "Your Zip Code"
}
~~~
If creating an Implementation Site was successful, an `Id` will be generated and the response will look similar to this:
~~~
{
    "id": "a0rDG000005N563TRE",
    "success": true,
    "errors": []
}
~~~

#### Find Program
Use the following API call to query the list of Programs to find yours. Use either `Type_of_Program__c='CDSME'` or `Type_of_Program__c='Falls+Prevention'` to filter on the CDSME and Falls Prevention program types respectively.
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select+Id+,+Name+FROM+epd_NCOA_Program__c+WHERE+Active__c='Active'+and+Type_of_Program__c='Falls+Prevention'
~~~
The response returns the `Id` and `Name` values for all the active programs based on the program filters. You will be able to select one from the list.

#### Find Program Target
Use the following API call to find your Program Target. Replace `Program__c+=+'XXXXXXXXXX'` with the Program Id from the previous step.
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select+id,+Funding_Source__c,+Program_Target_Search_TEXT__c,+Funding_Source_Name__C,+Program__c,+Record_Type_Name__c,+Survey_Template__c,+Survey_Template_TEXT__c+from+epd_Program_Target__c+where+Program__c+=+’XXXXXXXXXX’+and+Record_Type_Name__c+in+('Program+Specific','Collective') 
~~~
The Response gives ID, Funding source ID, Funding Source Name, Survey template ID and description details.
Select and make a note of the appropriate Funding source and Program target and make a note of the ID, Funding source ID for future steps.

#### Find Survey Template
Use the following API call to find your Survey Template. Replace `ID=’XXXXXXXXXX’` with the Survey Template Id from the previous step.
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select+id,+Name,+End_Date__c,+Program_Type__c,+Start_Date__c,+Template_Name__c+From+epd_Survey_Template__c+where+ID=’XXXXXXXXXX’+and+End_Date__c+=+null 
~~~
The Response gives the ID, Survey Template Name, Program type, Start Date of the survey template. Make a note of Survey template ID for Workshop creation.

#### Find Facilitators
In order to find the existing facilitators, Please use the below API : 
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select+id,+Contact__c,+First_Name__c,+Last_Name__c,+Email__c,+Active__c+FROM+epd_Facilitator__c+where+Active__C='Active'
~~~
The response gives the ID, Contact ID, First and Last name, Email ID . If the Facilitator is listed in the response, please select it or else, you can create a Contact and then a Facilitator as per Step 12.

#### Creating a Workshop
Create a Workshop:
 - Host Organization: Select the appropriate Host organization name. ID from Step 1 or 2.
 - Implementation Site: Select the appropriate Implementation name. ID from Step 3 or 4.
 - Program Type: Select respective Program type. Allowed values from below list.
 - Program Name: Select the appropriate Program name. ID from Step 5.
 - Program Delivery: Select respective Program delivery. Allowed values from below list.
 - Workshop start date: Start date in YYYY-MM-DD format.
 - Workshop end date: Start date in YYYY-MM-DD format.
 - Local name for the workshop (optional): Enter the local name (if applicable).
 - Workshop language: Select respective Language. Allowed values from below list.
 - Workshop language other: If language option is selected “Other (list)” then enter the value here.
 - Session 0: Select from Yes, No or Unknown.
 - Fee: Enter the Fee details.
 - Notes: Enter the Notes as needed. 
 - Number of sessions: Enter the no. of sessions.
 - Surveys: Select the appropriate Host organization name. ID from Step 7.

If the Facilitator exists in HAPI Database: 
After the Workshop is created, please proceed with adding the workshop details to epd_workshop_to_target__c and epd_Facilitator__c objects as per Steps: 9, 11.

If the Facilitator DOES NOT exist in HAPI Database:

After the Workshop is created, please proceed with [adding epd_workshop_to_target__c  objects](#updating-workshop-to-target-object).
Them, [Create a Contact](#create-contact) and [add workshop and contact details to epd_Facilitator__c](#create-facilitator).


API Call:
~~~
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_Workshop__c
~~~
Sample Request Body:
~~~
{
    "Host_Organization__c": "0013i000036abWdAAI", 
    "Implementation_Site__c": "a0r3i000003k1HoAAI",
    "Program_Type__c": "Falls Prevention",
    "NCOA_Program__c": "a0s3i00000BdcvqAAB",
    "Program_Delivery__c": "In-person", 
    "Workshop_Start_Date__c": "2023-10-01", 
    "Workshop_End_Date__c": "2023-10-04",
    "Local_name_for_this_workshop_optional__c": "Sample",
    "Workshop_Language__c": "English", 
    "Workshop_Language_Other__c": "", 
    "Session_0__c": "Yes",
    "Fee__c": "20",
    "Notes__c": "Thank you", 
    "Number_of_sessions__c": "2", 
    "Survey_Template__c": "a103i00000L6AayAAF"
}
~~~

#### Updating Worksop to Target Object
Update workshop details to Workshop to Target object.
Endpoint:
~~~
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_workshop_to_target__c
~~~
Sample Request Body:
~~~
{
    "Workshop__c": "a13DG0000089x7kYAA", 
    "Program_Target__c": "a0u3i00000BTUUcAAP"
}
~~~
Sample Response:
~~~
{
    "id": "a12DG00000Ks93FYAR",
    "success": true,
    "errors": []
}
~~~

#### Create Contact
Please use the below endpoint and request body to create a new contact.
~~~
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/Contact
~~~
Sample Request body:
~~~
{
    "Salutation": "Mr.", 
    "FirstName": "Maddy", 
    "MiddleName": "S", 
    "LastName": "Testing", 
    "Suffix": "Jr.", 
    "Email": "Maddy.testing@email.com", 
    "Title": "Facilitator", 
    "HasOptedOutOfEmail": "false", 
    "Employee_Type__c": "Volunteer", 
    "AccountId": "001DG00001d2li0YAA"
}
~~~
`Title` can always be defaulted on Facilitator.  `AccountId` is a Host organization account that was selected or created in Step 1 or Step 2. Allowed values for `Salutation` and `Employee Type` are listed in the [Appendix](#appendix).
~~~
Sample Response: 
{
    "id": "003DG00003pFR4NHGT",
    "success": true,
    "errors": []
}
~~~

Once after creating a new `Contact`, please proceed with creating a new `Facilitator`.

#### Create Facilitator
Once after creating a new Contact, please proceed with creating a new Facilitator. Please use the endpoint below and request body to create a new Facilitator.
Endpoint : 
~~~ 
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/Contact
~~~
Sample Request body:
~~~
{
    "Workshop__c": "a13DG0000089x7kYAA", 
    "Contact__c": "003DG00003pFSJmYAO", 
    "Active__c": "Active",
    "Employment_Type__c": "Volunteer"
}
~~~
Sample Response:
~~~
{
    "id": "a0nDG000004E6M9YAK",
    "success": true,
    "errors": []
}
~~~

## Participants
The Participant record is a record of an individual participant attending the Workshop session(s). As some Grantees record Participant workshop data against the Participant rather than the Workshop - the record will have a field that can optionally be used to store a non-identifiable external Id for the Grantee supplying the data. If they choose they can then match back to the Participant at a later date. Additionally the relation between Participant and Workshop record is managed through the Workshop Participant Object. 

The Participant record can not be deleted (unless a request goes to NCOA). The record can be marked as Active/Inactive. If a Participant or Participant data needs to be deleted, a support ticket should be raised and NCOA will delete as appropriate.

The Participant record will hold static survey data that does not change over time.

![Screenshot 2022-08-25 222216](https://github.com/djschlicht/hapid_api_documentation/assets/28845647/e2d83e18-231e-4815-a4c0-478d4c1e58e1)

### Creating a Participant


This is how to create a participant object.

### STEP1: Participants look up.   

Endpoint:  
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select+id+,+Name+,+Participant_Name_ID__c+,+from+epd_Participant__c+where+Participant_External_ID__c='NeededId'__ 
~~~
Replace the NeededId with the External Id of the participant which we need. 

  

Sample response:  

~~~
{  
  "id": "string",  
  "Name": "string",  
  "Participant_Name_ID__c": "string" 
}  
~~~
  
### STEP2: Adding a Participant.  



If the Participant does not exist in the Hapi database , you will be able to add the same.  

  

Endpoint:   
~~~
 POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_Participant__c/Participant_External_ID__c/{ExternalIdValue}
~~~
Replace the ExternalIdValue with the External Id of the participant. 
  

Sample requestbody:  

~~~
{  
  "id": "string",  
  "Name": "string",  
  "Participant_Name_ID__c": "string"  
}   
~~~
~~~
Sample Response:  
{ 
    "id": "001DG00001d2li0YER", 
    "success": true, 
    "errors": [] 
} 
 ~~~

#### STEP3: Survey Template lookup:  

Endpoint:   
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=select+id,+Name,+End_Date__c,+Program_Type__c,+Start_Date__c,+Template_Name__c+From+epd_Survey_Template__c+where+ID='XXXXXXXXXX'+and+Start_Date__c+<=+2023-09-01+and+(End_Date__c+>=+2023-09-30+OR+ENd_Date__c=null)  
~~~
Replace XXXXXXXXXX = Template ID from the selected workshop response.  


Select   

The Response gives ID, Survey template Name, Program type, Start Date of the survey template.  

Make a note of Survey template ID to Retrieve the Questions in the next step.  
~~~
{  
  "id": "string",  
  "Name": "string",  
  "Template_Name__c": "string",  
  "Start_Date__c": "string",  
  "End_Date__c": "string",  
  "Last_Workshop__c": "string",  
  "Program_Type__c": "string",  
  "Details__c": "string"  
}  
~~~  

#### STEP4:  Sections Lookup:  


Endpoint:  
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=SELECT+Section_Name__c +,+id+from+epd_Survey_Section__c+Where+Survey_Template__c+=+'Tempalte Id'  
~~~
  

Use the Template Id retrieved from the previous step  

  
The Response gives ID, Section Names.  

Select and make a note of the id for the section names that are needed. These will be used to retrieve question assignments and questions.  


Sample response:  
~~~
{  
  “Id”: “a0zDM000004O39LYAS”  ,
  "Section_Name__c": "Optional Items"  
}  
~~~
  

#### STEP5: Questions Lookup:  

Endpoint:  
~~~
GET https://ncoa1--uat.sandbox.my.site.com/services/data/v48.0/query?q=SELECT+id,+Question__r.API_Name__c,+Question__r.Custom_Label__c,+Question__r.Object_API_Name__c,+Question__r.Name,+Survey_Section__r.Section_Name__c+FROM+epd_Question_to_Survey_Section__c+WHERE+Survey_Template__c+=’TemplateID’ 
 ~~~
 

Use the Template Id retrieved from the above step  
  

The Response gives ID, API name, Object API Name and Label.  

Select and make a note of the Api Name of the field in which the answer is saved for the question, and the Api name of the Object where the field for the answer is located in. These both will help in retrieving the answers for the question for the given workshop participant.   

 ~~~
Response Body:  
{  
  “Id”: “string”  
  "API_Name__c": "string",  
  "Object_API_Name__c": "string",  
  "Custom_Label__c": "string", 
  “Name”: “string”, 
  “Section_Name__c”:”string”, 
  “Survey_Template__c”:”string” 
}  
 ~~~
 
 

#### STEP6: Saving the answers 

Initial Step to fill the answers  is to differentiate the questions from step8 according to the Object_API_Name__c 

Based on object API name we will have 3 different objects to save the answers as per step 7,8,9 

 
 
#### STEP7: Create or updating a Workshop Participant along with answers. 


If the Workshop  Participant does not exist in the Hapi database , you will be able to add the same.  

  ~~~ 
Endpoint:   
POST https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_Workshop_Participant__c/Import_ID__c/{WorkshopParticipantId}  
 ~~~
  

Replace the ExternalIdValue with the Import Id of the participant 

 
 

Use the Workshop__c value from the above 

Use the Participant__c from Step1or2 

USe the Survey_Template__c from workshop creation. 

Use the type_of_program__c from workshop creation. 

“Api name of Question to fill Answer”: “data type of the answer” 

 
Here Api name of question comes from step8 API_Name__c field “data type of the answer” is value of the answer to the question 

 
  

Sample requestbody:  

  
  
 ~~~
{ 

  "id": "string", 
  "Name": "string", 
  "Workshop__c": "string", 
  "Participant__c": "string", 
  "type_of_program__c": "string", 
  "survey_template__c": "string", 
  “Api name of Question to fill Answer”: “data type of the answer”, 
  “Api name of Question to fill Answer2”: “data type of the answer2”, 
  “Api name of Question to fill Answer3”: “data type of the answer3” 
  And so on as per the no of answers we have for the questions 
} 
  ~~~

Sample Response:  
 ~~~
{ 
    "id": "001DG00001d2li0YER", 
    "success": true, 
    "errors": [] 
} 
  ~~~
 
#### STEP8: Saving answers to participant object. 

 
 
 ~~~
https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_Participant__c/Participant_External_ID__c/{participantId}  

   ~~~

Replace the participantId with the External Id of the participant 

 
 

“Api name of Question to fill Answer”: “data type of the answer” 

 
Here Api name of question comes from step6 API_Name__c field and “data type of the answer” is value of the answer to the question 

  

Sample requestbody:  
 ~~~
{  
  "id": "string",  
  "Name": "string",  
  "Participant_Name_ID__c": "string" , 
  “Api name of Question to fill Answer”: “data type of the answer”, 
  “Api name of Question to fill Answer2”: “data type of the answer2”, 
  “Api name of Question to fill Answer3”: “data type of the answer3” 
  And so on as per the no of answers we have for the questions 
}  
  ~~~

Sample Response:  
 ~~~
{ 
    "id": "001DG00001d2li0YER", 
    "success": true, 
    "errors": [] 
} 
  ~~~

#### STEP9:  Saving answers to survey answer object. 


 ~~~
https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects/epd_Survey_Answer__c/Import_ID__c/{externalId}  
   ~~~

Replace the ExternalId with the Import_ID__c of the survey answer. 

“Api name of Question to fill Answer”: “data type of the answer” 

 
Here Api name of question comes from step8 API_Name__c field and “data type of the answer” is value of the answer to the question  
 ~~~
Sample requestbody:  
{ 
“Api name of Question to fill Answer”: “data type of the answer”, 
“Api name of Question to fill Answer2”: “data type of the answer2”, 
“Api name of Question to fill Answer3”: “data type of the answer3” 
And so on as per the no of answers we have for the questions 
}  

 
**Sample Response: **
{ 
    "id": "001DG00001d2li0YER", 
    "success": true, 
    "errors": [] 
}
~~~


#### STEP10: Get  Picklist values for the fields in the object 

 
 
~~~
https://ncoa1--uat.sandbox.my.site.com/services/data/v54.0/sobjects//{ObjectAPIName}/describe 
 ~~~
 

Here Object API name is the variable which describes the custom object one of three below 

 
 

Participant  - epd_Participant__c 

 
 

Survey answer - epd_Survey_Answer__c 

 

Questions - epd_Question__c 

[Participants fields](https://ncoa.sharepoint.com/:x:/s/Evidence-BasedProgramDatabase/Eeq7daL3_K9NrzUrRtozAsEBWjwauxBhlHVgoZkInfCM-w?e=x8V6LV)

All the fields pertaining to Workshop Participants , Participants and Questions objects can be referenced using the above link.


## Appendix
### Picklist Values
`Site_Type__c` can be one of the following:
 - Area Agency on Aging
 - County Health Department
 - Educational institution
 - Faith-based organization
 - Health care organization
 - Library
 - Multi-purpose Social Services Organization
 - Municipal government
 - Other
 - Other Community Center
 - Parks and Recreation/Other Recreational Organization
 - Residential facility
 - Senior center
 - State Health Department
 - State Unit on Aging
 - Tribal center
 - Workplace

`Program_Delivery__c` can be one of the following:
 - In-person 
 - Phone /teleconference 
 - Video-conference 
 - Mailed toolkits 
 - Self-directed 

`Workshop_Language_Other__c` can be one of the following:
 - English 
 - Spanish 
 - Arabic 
 - Bengali 
 - Chinese 
 - Dutch 
 - French 
 - German 
 - Greek 
 - Hindi 
 - Italian 
 - Japanese 
 - Korean 
 - Khmer 
 - Norwegian 
 - Punjabi 
 - Russian 
 - Somali 
 - Swedish 
 - Tagalog 
 - Tamil 
 - Turkish 
 - Vietnamese 
 - Other (list) 
 - American Sign Language 
 - Gujarati 
 - Navajo 
 - Portuguese 
 - Tongan 

`Salutation` can be one of the following:
 - Mr.
 - Ms.
 - Mrs.
 - Dr.
 - Prof.

`Employee_Type__c` to create a `Contact` can be one of the following:
 - Volunteer
 - Staff

`Employee_Type__c` to create a `Facilitator` can be one of the following:
 - Paid Staff Member

 - 
 - Volunteer
 - Other




