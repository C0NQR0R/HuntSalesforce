<!-- 
This template is Taken from this Research
https://www.enumerated.ie/index/salesforce#template 
-->

**Title:** [salesforce.site.com] Insecure Salesforce default/custom object permissions leads to information disclosure

* **Risk:** \*Low/Medium/High\*
* **Impact:** \*Low/Medium/High\*
* **Exploitability:** \*Low/Medium/High\*
* **CVSSv3:** \*CVSS_Score\* \*CVSS_STRING\*

**Target:** The Salesforce Lightning instance at `https://salesforce.example.com`.

**Impact:** The Salesforce Lightning instance does not enforce sufficient authorization checks when specific objects are requested. As such, an unauthenticated attacker may be able to extract sensitive data from the records in these objects which contains information of other users. This includes X,Y,Z in addition to other information.

**Description:** The web application at `https://salesforce.example.com` is built using [Salesforce Lightning](https://www.salesforce.com/eu/campaign/lightning/). Salesforce Lightning is a CRM for developing web applications providing a number of abstractions to simplify the development of data-driven applications. In particular, the [Aura](https://developer.salesforce.com/docs/component-library/bundle/aura:component) framework enables developers to build applications using reusable components exposing an API in order for the components to interact with the application.

During testing it was discovered that the Salesforce Lightning instance has loose permissions on the X,Y,Z objects for unauthenticated `Guest` users.

Therefore, a malicious attacker may be able to extract sensitive information belonging to other users of the application. To do this, an unauthenticated attacker may craft a HTTP request directly to the Aura API at `https://salesforce.site.com/s/sfsites/aura`, using built-in controller methods normally used by the Salesforce Lightning components. 

**Steps to Reproduce:**

1) Ensure Burp Suite is sniffing all HTTP(S) requests in the background
2) Navigate to `https://aaroncostello-developer-edition.eu45.force.com/`, this is to retrieve a template aura request for use
3) Find a POST request in Burp's Proxy history to the `/s/sfsites/aura` endpoint. Send it to the repeater
4) Modify both the `Host` header and Burp's target field to `salesforce.example.com`
5) Change the `message` POST parameter to the payload below. Please note that all other parameters should remain untouched, and that in this example payload, a pageSize of 100 is used for speed however more records can be retrieved:

```
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems","callingDescriptor":"UNKNOWN","params":{"entityNameOrId":"<OBJECT>","layoutType":"FULL","pageSize":100,"currentPage":0,"useTimeout":false,"getCount":false,"enableRowActions":false}}]}
```

6) Submit the request
7) The response contains sensitive information belonging to other users, an example screenshot has been provided below:

{{Screenshot}}

**Remediation:** Enforce [record level security (RLS)](https://help.salesforce.com/articleView?id=security_data_access.htm&type=5
) on the vulnerable object to ensure records are only able to be retrieved by the record owner, and privileged users of the application.
