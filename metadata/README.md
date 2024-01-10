# Metadata for HAPI-D's custom sObjects

It is often useful to look at the fields and other metadata associated with an sObject. To do so, use the sObject Describe API resource. Here is a link to the Salesforce documentation: [Get Field and Other Metadata for an Object](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_sobject_describe.htm).

The `json/` folder contains the responses of describe calls for custom sObjects associated with HAPI-D.

HTTP Example:
```http
GET https://ncoa1.my.salesforce.com/services/data/v59.0/sobjects/Account/describe/ -H "Authorization: Bearer token"
```

Python Example (using  [simple-salesforce](https://github.com/simple-salesforce/simple-salesforce)):
```Python
from simple_salesforce import Salesforce

sf = Salesforce (
  username='your-username',
  password='your-password',
  security_token='your-token',
  domain='login'
)

result = sf.Account.describe()
```

For information about what the many metadata values and fields represent: [DescribeSObjectResult](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_calls_describesobjects_describesobjectresult.htm).

## Contents of `json/`
- [epd_Facilitator__c](json/epd_Facilitator__c.json)
- [epd_Funding_Source__c](json/epd_Funding_Source__c.json)
- [epd_Grantee_to_Host_Organization__c](json/epd_Grantee_to_Host_Organization__c.json)
- [epd_Host_Org_To_Site__c](json/epd_Host_Org_To_Site__c.json)
- [epd_Implementation_Site__c](json/epd_Implementation_Site__c.json)
- [epd_NCOA_Program__c](json/epd_NCOA_Program__c.json)
- [epd_Participant__c](json/epd_Participant__c.json)
- [epd_Program_Target__c](json/epd_Program_Target__c.json)
- [epd_Program_to_Survey__c](json/epd_Program_to_Survey__c.json)
- [epd_Question__c](json/epd_Question__c.json)
- [epd_Question_to_Survey_Section__c](json/epd_Question_to_Survey_Section__c.json)
- [epd_Survey_Answer__c](json/epd_Survey_Answer__c.json)
- [epd_Survey_Section__c](json/epd_Survey_Section__c.json)
- [epd_Survey_Template__c](json/epd_Survey_Template__c.json)
- [epd_Workshop__c](json/epd_Workshop__c.json)
- [epd_Workshop_Participant__c](json/epd_Workshop_Participant__c.json)
- [epd_Workshop_To_Target__c](json/epd_Workshop_To_Target__c.json)


