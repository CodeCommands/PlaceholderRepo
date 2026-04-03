It is a very common scenario in Salesforce to run into deployment and coverage issues when retiring code, especially with the strict "do not delete triggers" rule many organizations enforce.
The issue you are facing happens because a Salesforce .trigger file must contain a valid declaration, and Salesforce requires every active trigger to have at least 1% code coverage. If you comment out the entire body of the trigger, it has zero executable lines, which causes the coverage validation to fail during deployment.
To safely retire the app, bypass the fflib framework, and satisfy Salesforce's coverage requirements, you need to replace the trigger's logic with a single "dummy" executable line and write a minimal test to cover it.
Here is the step-by-step process:
1. Update the Trigger with a Dummy Line
Do not comment out the entire trigger. Keep the trigger declaration intact, remove the fflib_SObjectDomain.triggerHandler(...) call, and add a single, harmless variable assignment. This provides an executable line for Salesforce to cover.
Before (Active fflib Trigger):
trigger MyObjectTrigger on MyObject__c (before insert, before update, after insert, after update) {
    // This is the line you are trying to retire
    fflib_SObjectDomain.triggerHandler(MyObjectsDomain.class); 
}

After (Retired Trigger):
trigger MyObjectTrigger on MyObject__c (before insert, before update, after insert, after update) {
    // App retired. Dummy line added strictly to satisfy the 1% trigger coverage rule.
    Integer retiredAppCoverage = 1; 
}

2. Create a Minimal Test Class for the Trigger
Since you commented out the old test classes, you need a tiny test class specifically designed to fire this trigger so that the Integer retiredAppCoverage = 1; line gets covered.
Create (or modify) a test class like this:
@IsTest
private class MyObjectTrigger_Test {
    @IsTest
    static void testRetiredTriggerCoverage() {
        // Create a dummy record to initiate a DML operation
        MyObject__c testRecord = new MyObject__c();
        
        // Populate any required fields here
        testRecord.Name = 'Coverage Test'; 
        
        Test.startTest();
        // Inserting the record fires the trigger and covers the dummy line
        insert testRecord; 
        Test.stopTest();
        
        // Simple assertion to ensure DML succeeded
        System.assertNotEquals(null, testRecord.Id, 'Record should insert successfully');
    }
}

3. Comment Out the Domain and Related Classes
Now that the trigger no longer references your fflib Domain class, you can safely comment out the entire body of those related classes.
You can select all the text inside the classes and use /* ... */ to comment them out. Because there is no longer any active code inside them, they will not impact your overall org code coverage.
 * Comment out: MyObjectsDomain.cls
 * Comment out: MyObjectsSelector.cls (if applicable)
 * Comment out: MyObjectsService.cls (if applicable)
 * Comment out: All original test classes related to the above.
Why this works:
 * Compliance: You haven't deleted the trigger, satisfying your org's rules.
 * Coverage: The trigger now has 1 executable line. Your new test class fires the trigger, resulting in 100% code coverage for that specific file.
 * Cleanliness: All the heavy fflib framework logic, selectors, and domain classes for the retired app are commented out, meaning they consume 0 resources and don't drag down your overall org coverage percentage.
