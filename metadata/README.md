# Metadata for HAPI-D's custom sObjects

For information about what the many metadata values and fields represent, please read the docs: [Salesforce Documentation](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_calls_describesobjects_describesobjectresult.htm).

The `.json` files found in `./json/*` were generated using [describeSObjects()](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_calls_describesobjects.htm) API calls (with [simple-salesforce](https://github.com/simple-salesforce/simple-salesforce) in Python).

Example:
```Python
import json
from simple_salesforce import Salesforce

# Login to Salesforce
sf = Salesforce (
  username='your-username',
  password='your-password',
  security_token='your-token',
  domain='login'
)

# describe() objects and write to file
result = sf.epd_Facilitator__c.describe()
with open('json/epd_Facilitator__c.json', 'w') as f:
  f.write(json.dumps(result, indent=2))
```

## Contents of `json/`
1. [epd_Facilitator__c](json/epd_Facilitator__c.json)
2. [epd_Funding_Source__c](json/epd_Funding_Source__c.json)
3. [epd_Grantee_to_Host_Organization__c](json/epd_Grantee_to_Host_Organization__c.json)
4. [epd_Host_Org_To_Site__c](json/epd_Host_Org_To_Site__c.json)
5. [epd_Implementation_Site__c](json/epd_Implementation_Site__c.json)
6. [epd_NCOA_Program__c](json/epd_NCOA_Program__c.json)
7. [epd_Participant__c](json/epd_Participant__c.json)
8. [epd_Program_Target__c](json/epd_Program_Target__c.json)
9. [epd_Program_to_Survey__c](json/epd_Program_to_Survey__c.json)
10. [epd_Question__c](json/epd_Question__c.json)
11. [epd_Question_to_Survey_Section__c](json/epd_Question_to_Survey_Section__c.json)
12. [epd_Survey_Answer__c](json/epd_Survey_Answer__c.json)
13. [epd_Survey_Section__c](json/epd_Survey_Section__c.json)
14. [epd_Survey_Template__c](json/epd_Survey_Template__c.json)
15. [epd_Workshop__c](json/epd_Workshop__c.json)
16. [epd_Workshop_Participant__c](json/epd_Workshop_Participant__c.json)
17. [epd_Workshop_To_Target__c](json/epd_Workshop_To_Target__c.json)


