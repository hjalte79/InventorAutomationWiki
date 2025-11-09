# Counting Checked-Out Files in Autodesk Vault with Power Automate

I’ve built a Power Automate Desktop flow that connects to Vault using its web services API, runs a search, and returns the results in a clean dialog. But that’s just the beginning.
This tutorial walks you through creating a modular flow that:

Authenticates with Autodesk Vault
Searches for checked-out files
Returns the results in a way that’s easy to reuse

The flow is divided into subflows for clarity and flexibility. I’ll guide you through each one step by step.
Since Power Automate lets you copy and paste flow logic as plain text, I’ll share the code for each subflow and explain what it does. This keeps the tutorial focused without overwhelming you with screenshots.
Once you have the results, you can take it further—send a Teams message, trigger an email, or integrate it into a larger automation. The possibilities are wide open.


# Step 1: Create the Main Flow

 Open Power Automate Desktop.
 
 Create a new flow and name it something like **Vault_Checkout_Status**.

![](./Images/CreateFlowCountCheckout.png)

Past the following code/logic:

```FlowLogic
CALL Setup
CALL login
CALL 'Lookup: KnowledgeVaultsId'
CALL 'Lookup property: Checked Out BY'
CALL Search
Display.ShowMessageDialog.ShowMessage Title: $'''Done''' Message: CheckedOutFiles Icon: Display.Icon.None Buttons: Display.Buttons.OK DefaultButton: Display.DefaultButton.Button1 IsTopMost: False
```

(Copy and past will not work if you have added extra returns in the logic.)

![](./Images/PastCode.png)

After pasting the logic “Power automate” will show you errors like this. 

![](./Images/MainCodeErrors.png)

This errors will get resolved during this tutorial.

# Step 2: Create the Setup Subflow
Add a new Subflow called “Setup”. 

![](./Images/CreateSubflow.png)

Past the following logic:

```FlowLogic
SET VaultServerUrl TO $'''http://XXXXXXXX'''
SET VaultName TO $'''XXXXXXXX'''
SET UserName TO $'''XXXXXXXX'''
SET Password TO $'''XXXXXXXX'''
```

In this subflow you will need to change the variables to your specific situation. The “VaultName”, “UserName” and “Password” can be taken from the login screen. However the “VaultServerUrl” needs to be tweaked. It should look something like this:
http://XXXXXXXX
You should replace the XXXXXXX with the Server name you can find in your vault client login screen. (Important: Don’t add a trailing slash) 

![](./Images/PaVaultLogin.png)

This information will be used to login to the vault. 

I expect that you can use the Vault gateway for the “VaultServerUrl”, however I have never tested that

The vault API also allows creating a login session using Windows credentials (Active Directory). That is not in the scope of this tutorial.

# Step 3: Create the “login” Subflow

Add a new Subflow called “login”. 

Past the following logic:

```FlowLogic
Web.InvokeWebService.InvokeWebServicePost Url: $'''%VaultServerUrl%/AutodeskDM/Services/api/vault/v2/sessions''' Accept: $'''application/json''' ContentType: $'''application/json''' RequestBody: $'''{
    \"input\": {
      \"vault\": \"%VaultName%\",
      \"userName\": \"%UserName%\",
      \"password\": \"%Password%\",
      \"appCode\": \"TC\"
    }
 }''' ConnectionTimeout: 30 FollowRedirection: True ClearCookies: False FailOnErrorStatus: False EncodeRequestBody: False UserAgent: $'''Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.21) Gecko/20100312 Firefox/3.6''' Encoding: Web.Encoding.AutoDetect AcceptUntrustedCertificates: False TrimRequestBody: False Response=> WebServiceResponse
Variables.ConvertJsonToCustomObject Json: WebServiceResponse CustomObject=> JsonAsCustomObject
SET BearerToken TO JsonAsCustomObject['accessToken']
```
([Reference Guide/sessions](https://aps.autodesk.com/en/docs/vaultdataapi/v2/reference/http/createsession-POST/))

As the name implies this subflow does the logging. The vault API returns lots of information but we will only use “accessToken” (or “BearerToken” as it is called in the documentation.) For all other calls to the vault API we need this token.

# Step 4: Create the “Lookup: KnowledgeVaultsId” Subflow
Add a new Subflow called “Lookup: KnowledgeVaultsId”. 

Past the following logic:

```FlowLogic
Web.InvokeWebService.InvokeWebService Url: $'''%VaultServerUrl%/AutodeskDM/Services/api/vault/v2/vaults''' Method: Web.Method.Get Accept: $'''application/json''' ContentType: $'''application/json''' CustomHeaders: $'''Authorization: Bearer %BearerToken%''' ConnectionTimeout: 30 FollowRedirection: True ClearCookies: False FailOnErrorStatus: False EncodeRequestBody: False UserAgent: $'''Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.21) Gecko/20100312 Firefox/3.6''' Encoding: Web.Encoding.AutoDetect AcceptUntrustedCertificates: False TrimRequestBody: False Response=> WebServiceResponse
Variables.ConvertJsonToCustomObject Json: WebServiceResponse CustomObject=> JsonAsCustomObject
LOOP FOREACH CurrentItem IN JsonAsCustomObject['results']
    IF CurrentItem['name'] = VaultName THEN
        SET KnowledgeVaultsId TO CurrentItem['id']
        DISABLE Display.ShowMessageDialog.ShowMessage Message: KnowledgeVaultsId Icon: Display.Icon.None Buttons: Display.Buttons.OK DefaultButton: Display.DefaultButton.Button1 IsTopMost: False
        EXIT FUNCTION
    END
END 
```
([Reference Guide/vaults](https://aps.autodesk.com/en/docs/vaultdataapi/v2/reference/http/getvaults-GET/))

This sub flow get the information of all vaults on your vault server. It loops over all the vaults and stops if it finds the vault with the given name in the “Setup”. Then it saves the “KnowledgeVaultsId” this is the identifier of the vault. This identifier is needed for a lot of call that you can make. In this case it will be used for the search API.

# Step 5: Get Property URL for "Checked Out By"
Add a new Subflow called “Lookup property: Checked Out BY”. 

Past the following logic:

```FlowLogic
Web.InvokeWebService.InvokeWebService Url: $'''%VaultServerUrl%/AutodeskDM/Services/api/vault/v2/vaults/%KnowledgeVaultsId%/property-definitions''' Method: Web.Method.Get Accept: $'''application/json''' ContentType: $'''application/json''' CustomHeaders: $'''Authorization: Bearer %BearerToken%''' ConnectionTimeout: 30 FollowRedirection: True ClearCookies: False FailOnErrorStatus: False EncodeRequestBody: False UserAgent: $'''Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.21) Gecko/20100312 Firefox/3.6''' Encoding: Web.Encoding.AutoDetect AcceptUntrustedCertificates: False TrimRequestBody: False Response=> WebServiceResponse
Variables.ConvertJsonToCustomObject Json: WebServiceResponse CustomObject=> JsonAsCustomObject
LOOP FOREACH CurrentItem IN JsonAsCustomObject['results']
    IF CurrentItem['displayName'] = $'''Checked Out By''' THEN
        DISABLE Display.ShowMessageDialog.ShowMessage Message: CurrentItem['url'] Icon: Display.Icon.None Buttons: Display.Buttons.OK DefaultButton: Display.DefaultButton.Button1 IsTopMost: False ButtonPressed=> ButtonPressed2
        SET CheckedOutPropertyURL TO CurrentItem['url']
        EXIT FUNCTION
    END
END
```
([Reference Guide/property-definitions](https://aps.autodesk.com/en/docs/vaultdataapi/v2/reference/http/getpropertydefinitions-GET/))

Searching in the vault is done base on properties. In this instance we want to find the property “Checked Out By”. In this subflow we get all properties know to the vault. Then we loop over all of them. Once the property “Checked Out By” is found we grab the url. This url is needed for the search that we want to perform.

# Step 6: Search

Add a new Subflow called “Search”. 

Past the following logic:

```FlowLogic
Web.InvokeWebService.InvokeWebServicePost Url: $'''%VaultServerUrl%/AutodeskDM/Services/api/vault/v2/vaults/%KnowledgeVaultsId%:advanced-search?limit=1''' Accept: $'''application/json''' ContentType: $'''application/json''' CustomHeaders: $'''Authorization: Bearer %BearerToken%''' RequestBody: $'''{
    \"EntityTypesToSearch\": [
        \"file\",
        \"folder\"
    ],
    \"FoldersToSearch\": [],
    \"SearchCriterias\": [
      {
        \"propertyDefinitionUrl\": \"http://skvnappp29%CheckedOutPropertyURL%\",
        \"operator\": \"IsNotEmpty\",
        \"searchString\": \"TRUE\"}
      
      ],
    \"SortCriterias\": null
  }''' ConnectionTimeout: 30 FollowRedirection: True ClearCookies: False FailOnErrorStatus: False EncodeRequestBody: False UserAgent: $'''Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.21) Gecko/20100312 Firefox/3.6''' Encoding: Web.Encoding.AutoDetect AcceptUntrustedCertificates: False TrimRequestBody: False Response=> WebServiceResponse
Variables.ConvertJsonToCustomObject Json: WebServiceResponse CustomObject=> JsonAsCustomObject
SET CheckedOutFiles TO $'''Found %JsonAsCustomObject['pagination']['totalResults']% checked out files.'''
```
([Reference Guide/advanced-search](https://aps.autodesk.com/en/docs/vaultdataapi/v2/reference/http/advancedsearch-POST/))

Now the fun part. We send a POST request to perform an advanced search. The criteria? Files where the "Checked Out By" property is not empty. Even though we limit the result to 1 item, the response includes the total count. We extract that and store a message like:

> Found 42 checked out files.

Finally, we show a dialog titled "Done" with the message from CheckedOutFiles. Simple, clean, and effective.
