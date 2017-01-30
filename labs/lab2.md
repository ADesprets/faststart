---
title: Lab 2 - Securing the Payments API using oAuth 2.0
toc: true
sidebar: labs_sidebar
permalink: /europe/2017/lab2.html
summary: In this lab, you’ll gain a high level understanding of the architecture, features and development concepts related to the IBM API Connect (APIC) solution. Throughout the lab, you’ll get a chance to use the APIC command line interface for creating LoopBack applications, the intuitive web-based user interface, and explore the various aspects associated with the solution’s configuration of RESTful based services as well as their operation. At the end of this lab you will have created an new application which provides access to product inventory via a set of API resource operations.
---

## Objective

In this lab you will learn how to secure an API using oAuth 2.0. The API you are securing is named 'Payments' and it has already been created for you. The instructions will guide you through how to create an oAuth 2.0 provider and then how to secure the 'Payments' API using the oAuth 2.0 provider you create. 

### What is oAuth 2.0?
Before we go further, let's take a look at the definition of oAuth 2.0:

"The OAuth 2.0 authorization framework enables a third-party
   application to obtain limited access to an HTTP service, either on
   behalf of a resource owner by orchestrating an approval interaction
   between the resource owner and the HTTP service, or by allowing the
   third-party application to obtain access on its own behalf."

## Components used in this Lab

The following diagram outlines the key interactions that we will be building in this lab.

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/arc_diag.png)

The components in this lab are:

- **The Payments API**: 
  - The Payments API is hosted and exposed by a financial institution who wants to allow third parties to make payments on behalf of its customers. 
  - The payments API is deployed on API Connect has 2 operations
      - The first operation is 'POST /payments' which allows a third party provider to initiate a payment on behalf of a user. Note this puts the payment in a 'pending' state and returns back a payment ID to the third party provider. This operation is secured by API keys only (no oAuth). 
      - The second  operation is 'POST /payments/{id}/execute' which allows the third party to finalise a payment and set it to an 'executed' state using the payment ID issued in the previous API call. This operation is secured by oAuth, therefore the user has to be redirected by the third party to the financial institution so they can sign in and confirm they authorize the third party to make the payment on their behalf. The oAUth flow handles the interaction between the user, third party provider and financial institution and it uses authorization codes and access tokens to delegate authorization from the user to the third party provider.  
   - The Payments API is provided for you in this lab. 
   
- **Authentication and Authorization Server**
   - This is the Authentication and Authorization Server (AAS) of the financial institution hosting the payments API. 
   - The user is redirected to the AAS of the financial institution to sign-in confirm they are happy to give the third party provider permission to make the payment. 
   - The user signs-in via the web interface of the AA and asked if they approve the payment they are shown on screen.
   - If the user confirms the AAS redirects them back to the third party via API Connect.
   - The AAS has been provided for you in this lab. 

- **The Payment Authorization (oAuth 2.0) API**: 
   - The Payment Authorization API (oAuth flow) handles the interaction between the user, third party provider and financial institution and it uses authorization codes and access tokens to delegate authorization from the user to the third party provider.  
   - 'Access Code' is the oAuth 2.0 flow used in the Payment Authorization API. The diagram below gives a detailed step by step guide of the access code oAuth flow. However, in summary, it invovles the user (client) being redirected to the AAS  and obtaining an authorization code. The authorization code is handed to the third party provider who in turns exchanges it for an access token which can then be used to make the payment. 
   - The oAuth flow has been enhanced slightly in this lab to include the passing of a payment ID in the oAuth flow. This is an important addition because in this scenario the user doesn't want to delegate authorization of a full 'scope' to the third party. The scope being used is 'payment approval' which means the user would be authorizing the third party to make ANY payment on their behalf. With the addition of the payment ID it means the user is authorizing the third party to make only the single payment being processed and not ANY payment. In this lab the payment ID is included in the URL as a parameter being passed between the different components. A more elegant and secure solution would be to enrich the access token to include the payment ID. API Connect has this capability and more information can be found on the Knowledge Center.
   
        [oAuth Metadata](https://www.ibm.com/support/knowledgecenter/en/SSMNED_5.0.0/com.ibm.apic.toolkit.doc/con_metadata.html) 
   
   - In this lab you will build the oAuth 2.0 provider API. 

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/accesscodeflow.png)

### Sequence of events

The sequence of events to make a payment is:

1. The third party calls the 'POST /payments' operation on behalf of the user to initiate the payment. The operation returns a payment ID to the third party.

2. The third party redirects the user to the Authentication and Authorization Server (passing the payment ID) to sign-in, approve the payment and obtain an authorization code.

3. The user (client) passes the authorization code to the third party, and the third party exchanges it for an access token (this is done by calling an operation exposed by the Payment Authorization (oAuth 2.0) API). 

4. The third party calls the POST /payments/{id}/execute operation passing the payment ID and access token to execute the payment

### Practical Notes
- In this lab you will be acting as the user, user client, financial institution and third party provider. 
- All the steps described above (e.g. sequence of events) you will be performing manually so you can understand how the flow and interactions work. 
- When running commands, particularly 'curl' commands, watch out for spaces and 'quotes'.
- Each section in the lab has a brief explanation of what you will be performing in that particular section. However, it's worth keeping in mind the wider picture of what you are building and relate each section back to it. 

## Part 1: Set up API Connect

This section can be skipped if you already have the API Connect Toolkit installed locally, and a Bluemix account with an API Connect service provisioned. If not, please follow the link below and complete ‘Lab 0 - Setup IBM API Connect’. Once complete, return to this document and move onto part 2. 

[Setup IBM API Connect Instructions](https://ibm-apiconnect.github.io/faststart/lab0.html) 

## Part 2: Secure and test the Payments API

### 2.1 Create a directory and clone the API Connect project from Github

This section will guide you through the set up of the base API Connect project on your laptop. You will need to have all the prerequisites specified in part 1 in order to perform these steps. 

1.	Navigate to a root folder on your laptop where you would like to keep your apic project
2.	Run the following command to create a directory

        mkdir madridApic
                        
3.	Change to the directory you just created by running the following command

        cd madridApic
                       
4.  Clone the API Connect project from github by running the following command

         git clone https://github.com/ibm-apiconnect/faststart-code


5.	Change to the directory you just created by running the following command

         cd faststart-code

6.	Run the following command to install the required NPM dependencies

          npm install 

### 2.2 Exploring the Payments API

This section will help you explore the payments API that is provided for you and the 2 operations it includes. The API is based on loopback and has a model named 'payment'. An in memory data store is used to store the data associated with the model. It is worth mentioning that if you have an API based on loopback then it has a Node.Js application (microservice) associated with it, as well as the API. This is an important consideration when it comes to deploying your API later, as you have to deploy the application as well as the API product. 

1.	Run the following command to start the API Connect toolkit:

        
        apic edit
        
2.	The toolkit will open up in your web browser and request you to sign into Bluemix, please do so using your IBM ID and password. 
3.	You will observe an API already exists named ‘payments’. This uses an in memory database to store details of payments and exposes a number of operations to allow payments to be reserved and later executed. 
 
 ![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-2-1.png)

4.	Click on the payments api to go into its design. 

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-2-2.png) 
        
        
5.	Scroll down until you can see the paths of the API. You will see 2 paths have been created for you:
 - a. POST / payments – this allows a payment reservation to be created and passes back a payment ID
 - b. POST payments/{id}/execute – this executes the payment and finalises it. It accepts a payment ID as a parameter in the URL (passed back from operation a). 
 
 ![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-2-3.png)

6.  Click on the 'All APIs' buton on the top left to return back to the main screen.


### 2.3 Creating the oAuth 2.0 Provider

This section will guide you through the creation and configuration of the oAuth 2.0 provider. The oAuth 2.0 provider will use the 'Access Code' flow and will use a redirect to the Authentication and Authorization Server of the financial institution to confirm the users identify and allow them to confirm they want to allow the third party to make payments on their behalf.

1.	Click on ‘Add’ and select ‘OAuth 2.0 Provider API’.

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-3-1.png)
        

2.	Name the oAuth API ‘payment authorization’ and select ‘Create API’.
 
![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-3-2.png)

        
3.	Scroll down to the oAuth 2 section and set:

        - Client Type: Confidential 
        - Scope Name: payment_approval (delete the other 2 scopes)
        - Grants: Check only ‘Access Code’, uncheck the others
        - Identify Extraction: Redirect
        - Redirect URL: https://aaservergk.eu-gb.mybluemix.net/login
        - Authentication URL: https://thinkibm-services.mybluemix.net/auth
        - TLS Profile: leave blank
        - Authorize application user using: Authenticated
        -Leave all other settings as default

     
![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-3-4.png)


4.	Click save on the top right.
5.	Scroll down until you see the section to declare parameters.
6.	Click the plus icon to create a new parameter, set:

        - Name: payment_id
        - Located in: Query
        - Required: yes (tick)
        - Type: String
 
 ![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-3-5.png)

7.	Click save on the top right.
8.	Scroll to the ‘paths’ section of the API. 
9.	Expand the GET /oauth2/authorize
10.	Click ‘Add Parameter’ and select ‘payment_id’.

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab2/2-3-6.png)
           
11.	Click save on the top right.

### 2.4 Securing the Payments API with the oAuth Provider

This part of the lab will walk you through the steps required to secure the payments API using oAuth 2.0 provider you created in the previous section. 

   - a. POST / payments – this operation is secured with a client secret and ID only since it only initiates the payment (i.e. places it in a pending state). 

   - b. POST payments/{id}/execute – this operation is secured with oAuth 2.0 (since it’s the operation which executes the payment and finalises the financial transition). 
   
1.	Press ‘All APIs’ at the top to return to the list of APIs
2.	Click on the payments API to go in and see the details. 
 
     <img src="/madridapiclab2/images/2-4-1.png" width="450">

3.	Scroll down to the ‘Security Definitions’ section and click the + icon to create a new security definition. Select oAuth, then set:

        - Name: payment approval
        - Flow: Access Code
        - Authorization URL: see below
        - Token URL: see below
        - Add a scope using the (+), scope name: payment_approval


        ````
        Authorization URL is made up of:

        https://<bluemix host region url>/<organization-space_name>/<catalog_name>/payment-authorization/oauth2/authorize

        For example, 
                United Kingdom Bluemix URL: api.eu.apiconnect.ibmcloud.com
                Organization: gary.kean@uk.ibm.com
                Space: APIConnect
                Catalog: workshop-demo

        the Authorization URL would look like this: 
        https://api.eu.apiconnect.ibmcloud.com/garykeanukibmcom-apiconnect/workshop-demo/payment-authorization/oauth2/authorize

        Token URL is made up of:

        https://<bluemix host region url>/<organization-space_name>/<catalog_name>/payment-authorization/oauth2/token

        For example,
                United Kingdom Bluemix URL: api.eu.apiconnect.ibmcloud.com
                Organization: gary.kean@uk.ibm.com
                Space: APIConnect
                Catalog: workshop-demo

        the Token URL would look like this: 
        https://api.eu.apiconnect.ibmcloud.com/garykeanukibmcom-apiconnect/workshop-demo/payment-authorization/oauth2/token

        You can double check these URLs in the developer portal later once you have published your API.
        ````


   <img src="/madridapiclab2/images/2-4-2.png" width="450">
        
        
        
4.Scroll down to the ‘Security’ section of the API and you will see that the client ID and the client Secret are the default security measures added to each operation of the API unless configured otherwise. We therefore have to specifically add the oAuth 2.0 provider we created to the execute payment operation. Ensure the ‘Security’ section of your API matches the screen shot below.  
 
   <img src="/madridapiclab2/images/2-4-3.png" width="450">

5.Scroll down to the ‘Paths section and expand the path for ‘POST payments/{id}/execute’.

6.Ensure the security options for the API does not use the ‘API security definitions’ and instead is set to have the ‘payment approval’ oAuth  (note the client ID and client secret should also be unset). 
 
   <img src="/madridapiclab2/images/2-4-5.png" width="450">

7.Click save on the top right.

### 2.4 - Launch the API Connect Designer

1.  Ensure you're in the `~/ThinkIBM/inventory` directory, then type the following command:

    ```shell
    apic edit
    ```

    The Firefox web browser will launch and automatically load the designer screen.

1.  Now that the API Designer is running, you should see the start page with your `inventory` API.

    {% include note.html content="This API was created as a result of the generation of our LoopBack application.
    " %}

### 2.5 - Create a Model for the Inventory Items

In this section, you will define the `item` data model for our `inventory` API and attach it to the Cloudant data source. LoopBack is a data model driven framework. The properties of the data model will become the JSON elements of the API request and response payloads.

1.  Click the `Models` tab.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/api-designer-model-design-page.png)
	
1.  Click the `+ Add` button.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/api-designer-model-design-page-add-button.png)
	
1.  In the New LoopBack Model dialog, enter `item` as the model name, then click the `New` button.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/api-designer-model-design-page-new-model.png)

1.  When the Model edit page for the item model displays, select the `item-db-cloudant` Data Source:

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/1.png)

### 2.6 - Create Properties for the `item` Model

The item table in the database has 6 columns that will need to mapped as well. To start creating properties for the item model: 

1.  Click the `+` button in the **Properties** section.

	![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/properties26.png)
	
1.  The `item` data model consists of six properties. Use the data below to add each of the properties:

    |Required|Property Name|Is Array|Type  |ID |Index|Description           |
    |--------|-------------|--------|------|---|-----|----------------------|
    |yes     |name         |no      |string|no |no   |item name             |
    |yes     |description  |no      |string|no |no   |item description      |
    |yes     |img          |no      |string|no |no   |location of item image|
    |yes     |img_alt      |no      |string|no |no   |item image title      |
    |yes     |price        |no      |number|no |no   |item price            |
    |no      |rating       |no      |number|no |no   |item rating           |

1.  Scroll to the top of the page and click the `Save` button to save the data model.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/api-designer-model-design-page-model-properties-save.png)

1.  Click the `All Models` link to return to the main API Designer page.

### 2.7 - Verify API

To confirm that the API has been correctly mapped and can interface with the datasource, you will run the server and test the API.

1.  Click the `Run` button to start the `inventory` LoopBack application

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/run.png)

1.  Wait a moment while the servers are started. Proceed to the next step when you see the following:

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/app-running.png)

1.  Click the `Explore` button to review your APIs. 

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/explore.png)

1.  On the left side of the page, notice the list of paths for the `inventory` API. These are the paths and operations that were automatically created for you by the LoopBack framework simply by adding the `item` data model. The operations allow users the ability to create, update, delete and query the data model from the connected data source.

1.  Click the `GET /items` operation.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/api-designer-explore-page-get-items-api.png)

1.  By clicking the `GET /items` operation, your screen will auto-focus to the correct location in the window. In the center pane you will see a summary of the operation, as well as optional parameters and responses.

    On the right side you will see sample code for executing the API in various programming languages and tools such as cURL.

    In addition to the sample code, if you look further down the page you will see an example response, URL, API identification information, and API parameters.

1.  Scroll down slowly to locate the `Call operation` button.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/api-designer-explore-page-call-operation.png)

1. Click the `Call operation` button to invoke the API.

    {% include troubleshooting.html content="The first time you invoke the API, you may receive an error. The error occurs becuase the browser does not trust the self-signed certificate from the MicroGateway. To resolve the error, click on the link in the response window and accept the certificate warning.
    " %}

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/cert-error.png)

1.  Once complete, return to the API explorer and click on the `Call operation` button again.

1.  Scroll down to see the `Request` and `Response` headers. 

    ```text
    Request
    GET https://localhost:4002/inventory/items
    APIm-Debug: true
    Content-Type: application/json
    Accept: application/json
    X-IBM-Client-Id: default
    X-IBM-Client-Secret: SECRET
    ```

    ```text
    Response
    Code: 200 OK
    Headers:
    content-type: application/json; charset=utf-8
    x-ratelimit-limit: 100
    x-ratelimit-remaining: 99
    x-ratelimit-reset: 3599999
    ```

1.  Scroll further and the payload returned from the GET request is displayed.

    ```json
    [
      {
        "name": "Dayton Meat Chopper",
        "description": "Punched-card tabulating machines and time clocks...",
        "img": "images/items/meat-chopper.jpg",
        "img_alt": "Dayton Meat Chopper",
        "price": 4599.99,
        "rating": 0,
        "id": 5
      },
      ...
    ]
    ```

1.  Test the `GET /items/count` operation by following the same process above. You should receive a count of inventory items.

    ```json
    {
      "count": 12
    }
    ```

1.  Click on the `Stop` button to shut down the running application.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/stop-application.png)

1.  Click on the `X` in the top-right portion of the screen to leave the API Explorer view.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/leave-explorer.png)

### 2.8 - Create the second Cloudant Data Source for Item Reviews

So far, we have created a LoopBack application which provides APIs around our inventory items stored in a Cloudant database in Bluemix.

In the next section, you will create the data model for item reviews which will use Cloudant to store the review data.

First you must create a data source entry for the Cloudant Reviews DB.

In the earlier steps, you used the command line to create a data source connection. This time you will use the API Designer.

1.  From the top navigation menu, select the `Data Sources` link to switch views.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/data-sources.png)

1.  Click on the `Add +` button.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/add-new-datasource.png)

1.  Name the new datasource `review-db-cloudant` and click the `New` button.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/add-new-datasource-name.png)

1.  Complete the new datasource configuration using the values in the table below.

    |Field      |Value        |
    |-----------|-------------|
    |URL        |https://820923e0-be08-46f5-a34a-003f91f00f5c-bluemix:10d585c237c8d7b599b79cfcca39cb63356f2cea7d79abf27f284801b3c149d9@820923e0-be08-46f5-a34a-003f91f00f5c-bluemix.cloudant.com|
    |Database   |review       |
    |Username   |<leave blank>|
    |Password   |<leave blank>|
    |Model Index|<leave blank>|

1.  Click on the `Save` icon to save the new data source connection. The toolkit will test the connection and report back. 

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/add-new-datasource-success.png)

### 2.9 - Create Model for Reviews

The `review` data model will be used to store item reviews left by buyers. The reviews will be stored in a Cloudant.

In the earlier steps, you used the API Designer User Experience to create a data model. This time you will use the command line to create the `review` model.

1.  Click the `x` button on your browser tab to close the API Designer.

1.  Select the `Terminal Emulator` from the taskbar to open the command line.

1.  Even though we closed the browser, the API Designer application itself is still running.

    Hold the `control` key and press the `c` key to end the API Designer session:
	
    ```shell
    control+c
    ```
	
    This will return you to the command line prompt.

1.  Type the following command to create the `review` data model:

    ```shell
    apic create --type model
    ```

1.  Enter the properties for the `review` model:

    {% include important.html content="You will **not** expose the review mode as a REST API. This is because you create a relationship between item and review later that will create the REST APIs you will use.
    " %}

    ```text
    ? Enter the model name:  review
    ? Select the data-source to attach review to:
    	> review-db-cloudant (cloudant)
    ? Select models base class:
    	> PersistedModel
    ? Expose review via the REST API? (Y/n):  N
    ? Common model or server only?
    	> common
    ```
	
1.  Continue using the wizard to add properties for the `review` model:

1.  The first property is the `date` property:

    ```text
    Enter an empty property name when done.
    ? Property name: date
    ? Property type:
    	> date
    ?Required? Y
    ?Default value [leave blank for none]: <leave blank>"
    ```

1.  Next add the `reviewer_name` property:

    ```text
    Let's add another review property.
    Enter an empty property name when done.
    ? Property name: reviewer_name
    ? Property type:
    	> string
    ? Required? N
    ? Default value [leave blank for none]: <leave blank>
    ```

1.  Next add the `reviewer_email` property:

    ```text
    Let's add another review property.
    Enter an empty property name when done.
    ? Property name: reviewer_email
    ? Property type:
    	> string
    ? Required? N
    ? Default value [leave blank for none]: <leave blank>
    ```

1.  Next add the `comment` property:

    ```text
    Let's add another review property.
    Enter an empty property name when done.
    ? Property name: comment
    ? Property type:
    	> string
    ? Required? N
    ? Default value [leave blank for none]: <leave blank>
    ```

1.  Finally add a property for the item `rating`:

    ```text
    Let's add another review property.
    Enter an empty property name when done.
    ? Property name: rating
    ? Property type:
    	> number
    ? Required? Y
    ? Default value [leave blank for none]: <leave blank>
    ```

1.  To close the wizard, the next time it asks you to add another review proporty, just press `Enter` or `Return` to exit.

### 2.10 - Create a Relationship Between the `item` and `review` Data Models

The next step in this lab is to create a relationship between the `item` model and the `review` model. Even though the models reference entities in entirely different databases, API Connect provides a way to create a logical relationship between them. This logical relationship is then exposed as additional operations for the item model.

1.  In the terminal session, type the following command:

    ```shell
    apic loopback:relation
    ```

1.  Enter the details for the relationship as follows:

    ```text
    ? Select the model to create the relationship from:
    	> item
    ? Relation type:
    	> has many
    ? Choose a model to create a relationship with:
    	> review
    ? Enter the property name for the relation:  reviews
    ? Optionally enter a custom foreign key: <leave blank>
    ? Require a through model? No
    ```
	
### 2.11 - Verify the Relationship

To verify that the relationship has been created, you will open the API Connect Designer and view the operations on the Explore page.

1.  In the terminal session, type the following command to launch the API Connect Designer window:

    ```shell
    apic edit
    ```

1.  Click on the `inventory` link from the APIs tab.

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/inventory-link.png)

1.  Scroll down to the `Paths` section of the API definition.

    Notice how three new API paths have been created which allow access to item revew data:

    ![](https://github.com/ibm-apiconnect/pot/raw/gh-pages/images/lab2/new-paths.png)

1.  Click the `x` button on your browser tab to close the API Designer.

1.  Select the `Terminal Emulator` from the taskbar to open the command line.

1.  Even though we closed the browser, the API Designer application itself is still running.

    Hold the `control` key and press the `c` key to end the API Designer session:
	
    ```shell
    control+c
    ```
	
    This will return you to the command line prompt.

## Conclusion

**Congratulations!** In this lab you learned:

+ How to create a multi-model LoopBack application
+ How to create a Representational State Transfer (REST) API definition using IBM Connect API Designer
+ How to create a Representational State Transfer (REST) API definition using IBM Connect Command Line
+ How to use the LoopBack Cloudant Connector
+ How to test a REST API
+ How to create relationships between models

Lab 3 will build on what you have already created to enable processing hooks and publish the APIs to the API Manager.

Proceed to [Lab 3 - Customize and Deploy an Application](lab3.html)

