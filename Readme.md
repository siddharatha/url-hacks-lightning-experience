URL hacks in Lightning Experience
=
This is a prototype to solve the url hacks in Lightning Experience.

Lightning Experience is forcing us to get rid of the custom buttons on related lists to prefill values , and I believe there is a design decision behind this to rethink our implementations as Single Page Applications and offer better UI without leaving the page, but from an enterprise implementation URL hacks was way to easy to prefill information and the ability to call it from anywhere is extremely convinient . May be am wrong , I would like to hear more thoughts or ideas on this.


The approach I wanted to initially pursue the minute I saw the 
```html
<Force:inputField value="{!v.sobject.fieldname}"/>
```
was yeah, lemme find a way to call this dynamically create a form by using field sets .

[Stack overflow - Lightning from FieldSets](http://salesforce.stackexchange.com/questions/152590/lightning-component-dynamically-create-form-and-submit-need-help)

but , lightning component documentation says that

> Creating components where the top-level components don’t have server dependencies but nested inner components do is not currently supported.

It wouldn't work if I try to create the component dynamically. This was so close .I actually thought we can dynamically recreate the UI . but it failed after throwing errors.

Quick Action
=
Quick action works, if I am launching it from my record home page , but if i want this functionality on my existing custom visualforce page which is propogating tons of dynamic links I couldn't find an alternative with the QuickAction API . 

But it gave some ideas of getting the layout definition without relying on fieldsets . But it would be better to have some alternative where we can replicate the UI without creating quick actions also.

[Lightning API](https://developer.salesforce.com/docs/atlas.en-us.lightningapi.meta/lightningapi/)
=
Lightning API is cool and its light weight. I can get the layout definition, which fields to show on create, a easier way to get the picklist definitions.

```curl
    /lapi/record-defaults/create/{apiName}
```
was the one that I absolutely needed , because majority of my cases are new record creation, and am sure if I find a solution with this , I can find a workaround for edit record.

Visualforce
=
Visualforce is still the only way for us to open stuff from related list - list buttons . Which is a common use case in my company.

Things we need
=
* Generic Visualforce page
* CORS
    * yourdomain.lightning.force.com
    * yourdomain.visual.force.com
    * yourdomain.my.salesforce.com
* RemoteSite
    * yourdomain.lightning.force.com
    * yourdomain.visual.force.com
    * yourdomain.my.salesforce.com
* CreateRecord - Lightning Component
* InputField - Lightning Component
* dependency Lightning Components
    * lightning:input
    * lightning:select

Recipie
=
* Prepare the base visualforce page
    * add ``` <apex:slds /> ```
    * parse url parameters
        * Naming convention for URL parameters        
            * retUrl - lets parse the info from this
            * record_fieldName_fieldValue
                - lookup fields: record_AccountId_value=001AA..., record_AccountId_label=Test%2CAccount
                - picklist fields: record_Status_value
                - text/number fields: record_randomtext_value
    * Minimize the use of external javascript libraries as your page is gonna be a bit slow anyways, so why add more. but if its from some cdn or cached , go ahead, but benchmark it in your company aproved browsers and test.
    * use the Fetch function to make the API call to ```lapi ``` and the result is returned as 3 parts , it seems like its provided this way to make us rebuild the UI anywhere we want. Cool move by salesforce.
        - layout
        - record
        - objectInfo
    * Subsequent picklist definitions should be retrieved again. So for a bunch of API calls, we better use promise api. I am directly gonna use the Promise and Fetch , its not still supported in IE 11 but there are some polyfill libraries il add links to.