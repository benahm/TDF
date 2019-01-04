
## Limitations 


#### Predefault values for picklist/mulipicklist fields 


In case of an sObject with multiple record types, TDF will assign the default picklist value and if the default value is not visible for the used recrod type, you will get the error below

*Example: Contact has multiple record types and the field CustomField__c the 'TEST' value as default, but it's not assigned to the record types*
	
    TDF.TestDataFactoryException: Unable to insert "Contact" records: 
    INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST: PicklistExample: bad value for restricted picklist field: TEST [CustomField__c=TEST]
    Stack Trace 

    Class.TestDataFactory.SObjectManager: line 381, column 1
    Class.TestDataFactory.SObjectManager.insertAll: line 302, column 1
    Class.TestDataFactory.SObjectFactory: line 465, column 1
    Class.TestDataFactory: line 149, column 1
    Class.TestDataFactory: line 91, column 1
    Class.TestDataFactory: line 76, column 1
    Class.WS_ScoreIntegration_Test.testSetup: line 44, column 1

This restriction is due to Apex limitation [no way to get picklist values based on Record Type](https://success.salesforce.com/ideaView?id=08730000000gNpLAAU) 

###### Workaround

Define a custom ``TDF.DefaultValueProvider`` implementation and handle the picklist default value per sObject & field


  ```apex
public class TestDefaultValueProvider extends TDF.DefaultValueProvider{
    public override String getPicklistDefaultValue(Schema.DescribeSObjectResult sObjectDesc, Id recordTypeId, Schema.DescribeFieldResult fieldDesc, Integer recordIndex){
      Map<String,Schema.RecordTypeInfo> recordTypesMap = sObjectDesc.getRecordTypeInfosByDeveloperName();
      if(sObjectDesc.getSObjectType() == Contact.SObjectType){
        if(fieldDesc.getName() == 'CustomField__c'){
          if(recordTypesMap.get('SomeRecordType').getRecordTypeId() == recordTypeId ){
            return 'Value Assigned to the Record Type';
          }
        }
      }
      return getSFDefaultPicklistValue(fieldDesc);
    }
  }
  ```