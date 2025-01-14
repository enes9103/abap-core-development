---
parser: v2
auto_validation: true
time: 20
tags: [ tutorial>beginner, software-product>identity-authentication, topic>cloud, tutorial>license]
primary_tag: software-product>sap-btp--abap-environment
---

# Provision Users into your SAP BTP ABAP Environment
<!-- description --> Provision and authorize users for ABAP development via Cloud Identity Services in one or more target systems

## Prerequisites
 - You have installed and set up Eclipse with the ABAP Development Tools (ADT) plugin, see <https://tools.hana.ondemand.com/#abap>  
 - You have an **SAP Business Technology Platform** customer subaccount and have prepared the following
    - Subscription to**Cloud Identity Services**
    - Established trust to your **SAP Cloud Identity Services - Identity Authentication tenant**
    - Created an **ABAP environment** service instance for custom development with
        - An SAP Fiori launchpad business role for custom ABAP development created from template `SAP_BR_DEVELOPER`
        - A service key for ADT integration
 - You have one or more users with authorization for
    - User and Group Management in your **SAP Cloud Identity Services - Identity Authentication tenant**
    - Space development in your ABAP system service instance's Cloud Foundry space

## You will learn
  - How to create and group developer users in your **SAP Cloud Identity Services - Identity Authentication tenant** (IAS)
  - How to enable Identity Provisioning in your IAS
  - How to configure and run Identity Provisioning
  - How to connect Eclipse with the ABAP environment

## Intro
Additional information:

  - In this use case the **SAP Cloud Identity Services - Identity Authentication tenant** is used as an identity provider and not as a proxy to another identity provider.
  - [Documentation: SAP Cloud Identity Services – Identity Provisioning](https://help.sap.com/viewer/f48e822d6d484fa5ade7dda78b64d9f5/Cloud/en-US/2d2685d469a54a56b886105a06ccdae6.html)

---

### Create User in IAS Tenant

As first step you create the future development user's identity on your IAS.

Logon with your User Management Administrator to your Identity Authentication tenant's administration UI (URL ends with path `/admin`, for example https://rapworkshop.accounts.ondemand.com/admin).

Navigate to **Users & Authorizations** > **User Management**

![Open User Management in IAS](IAS_user_mgmt_open.png)

Select **Add User** to start the creation process of a user.

![Press Add button for new user](IAS_user_add.png)

Fill the personal information for the user and select **Save**

![Configure properties of new user](IAS_user_configure_new.png)

The new user is now displayed in the list of users.

![List entry for new user](IAS_user_created.png)

>Note that the user will receive an e-mail to activate the account before being able to logon for the first time.


### Create Developer Group and Assign User

To bundle developers users, create a corresponding user group in the IAS and assign the users to it.

Navigate to **Users & Authorizations** > **User Groups** and select **Create**

![Start User Group creation](IAS_group_create.png)

In the Create Group dialog enter a **Name** and **Display Name** and select **Create**
> Note that the group name has to be the same as the one you have chosen for the business role (created from template `SAP_BR_DEVELOPER`, see prerequisites) in your ABAP environment.

![Configure properties of new group](IAS_group_configure_new.png)

To add users to the group select **Add**.

![Start user adding to group](IAS_group_add_user.png)

Search for the user that you have created earlier, select it, and choose **Save**

![Save User Group](IAS_group_add_user_save.png)

The user is now displayed in the user group list.

![List entry for user in group](IAS_group_add_user_result.png)



### Authorize Identity Provisioning Manager

Authorize an Administrator user for Identity Provisioning Management.

Navigate to **Users & Authorizations** > **Administrators** choose the Administrator user, slide the toggle button for **Manage Identity Provisioning** to **ON** and select **Save**

![Set Identity Provisioning Management](IAS_IPS_admin_configure.png)


### Configure Access to Source via technical user

In this example the IAS itself is used as a source for users and user groups that can be provisioned to other systems.
To allow identity provisioning to read users and groups from the IAS, you need a technical user with corresponding permissions.

Navigate to **Users & Authorizations** > **Administrators**

Select **Add** and choose **System**

![Start Administrator creation](IAS_admin_add.png)

Provide a **Name** for the system, for example `ips_tutorial_admin`.
Make sure to only set authorizations for **Read Users** and **Manage Groups** which are both needed to read users and groups during identity provisioning, **Save** your changes.

![Configure Administrator Authorizations](IAS_admin_configure.png)

A new dialog to **Configure User ID and Password** is displayed. Set the password and select **Save**. The User ID is generated automatically.

![Set Password](ias_admin_set_pw.png)

Go back to the **Configure User ID and Password** screen and **Copy** the generated **User ID** of the technical user.

![Copy Technical User's ID](IAS_admin_copy_user_id.png)


### Represent Source in Identity Provisioning

Identity provisioning requires to configure a so-called source system. This system represents where the user and user group data shall be gathered from that can be provisioned to other systems.

Logon with your Identity Provisioning Manager user to your Identity Authentication tenant's identity provisioning UI (URL ends with path `/ips`, for example https://rapworkshop.accounts.ondemand.com/ips).

Select the **Source Systems** tile

![Source System Tile](ips_source_systems_tile.png)

To start the creation, select **Add**

![Add Source System button](ips_source_system_add.png)

To simplify the system creation and to reduce the risk of errors, this tutorial provides a template JSON file for the source system. Download [`ips_system_template_source.json`](https://raw.githubusercontent.com/SAPDocuments/Tutorials/master/tutorials/abap-env-ips/ips_system_template_source.json) locally.

Define the system by uploading the JSON file via **Browse** in the IPS system UI.

![Browse for source system template file](ips_source_systems_browse.png)

Adapt the values to your needs and provide the mandatory values for `URL`, `User` and `Password` as shown below.

Alternatively, you can configure everything manually.

Details:

|  Label     | Value
|  :------------- | :-------------
|  Type           | Identity Authentication
|  System Name           | For example **`My IAS ABAP Developers`**

Properties:

|  Name     | Value
|  :------------- | :-------------
|  **`Type`**           | **`HTTP`**
|  **`ProxyType`**           | **`Internet`**
|  **`URL`**          | your IAS URL, for example <https://rapworkshop.accounts.ondemand.com>
|  **`Authentication`** | **`BasicAuthentication`**
|  **`User`**    | User ID of technical user
|  **`Password`**   | Password of technical user
|  **`ias.user.filter`**   | **`groups.display eq "BR_IPS_TUTORIAL_DEVELOPER"`**
|  **`ias.group.filter`**   | **`displayName eq "BR_IPS_TUTORIAL_DEVELOPER"`**

**Save** your changes.


### Configure access to Target

To enable identity provisioning to create users and assign business roles in another system, that system has to provide the corresponding authorization to identity provisioning.

If you have used an ABAP environment instance, navigate to **Services > Service Instances** in your SAP BTP subaccount and select **Create Service Key**
![Start Service Key creation for ABAP instance ](btp_service_key_create.png)

In the **New Service Key** dialog, provide a name and copy & paste the following JSON code.

```JSON
{
  "scenario_id":"SAP_COM_0193",
  "type":"basic"
}
```
**Create** the new service key
![Configure new service key](btp_service_key_configure_new.png)



>This service key creation automatically creates a communication user (1), communication system (2) and communication arrangement (3) for communication scenario `SAP_COM_0193` (4) in the ABAP environment instance.
![Communication Artefacts for IPS in ABAP instance](ABAP_FLP_CA_created.png)
Communication scenario `SAP_COM_0193` exposes all the needed services for identity provisioning. With the communication user credentials, you can make inbound calls to that system to provision groups and users.

The credentials created for the communication user were also transferred to the subaccount. To view the credentials of the communication user in your subaccount, navigate to **Services > Instances and Subscriptions** and select your service instance.

![Open Service Instance Details](BTP_ABAP_env_view_details.png)

From the Actions menu, choose **View Credentials**.

![View Credentials of service instance](BTP_ABAP_env_view_credentials.png)

Choose the credentials that you have set earlier in the service key for IPS usage.
Copy the **`username`**, **`password`** and **`url`** value for the next step.     

![Get communication user credentials from service key](btp_service_key_get_credentials.png)


### Represent Target in Identity Provisioning

Identity provisioning requires to configure a so-called target system. This system represents where the user and user group data shall be provisioned to.
In this example, the target systems is an ABAP system in SAP BTP.

Logon with your Identity Provisioning Manager user to your Identity Authentication tenant's identity provisioning UI (URL ends with path `/ips`, for example <https://rapworkshop.accounts.ondemand.com/ips>).

Select the **Target Systems** tile

![Source Target Tile](ips_target_systems_tile.png)

To start the Creation, select **Add**

![Add Target System button](ips_target_system_add.png)

To simplify the system creation and reduce the risk of errors, this tutorial provides a template JSON file for the source system. Download [`ips_system_template_target.json`](https://raw.githubusercontent.com/SAPDocuments/Tutorials/master/tutorials/abap-env-ips/ips_system_template_target.json) locally.

Define the system by uploading the JSON file via **Browse** in the IPS System UI.

![Browse for target system template file](ips_target_systems_browse.png)

Adapt the values to your needs and provide the mandatory values for `URL`, `User` and `Password` as shown below.

Alternatively, you can configure everything manually.

Details:

|  Label     | Value
|  :------------- | :-------------
|  Type           | SAP BTP ABAP environment
|  System Name           | For example **`My ABAP instance`**
|  Description           | For example **`System to receive provisioned Developer Users`**
|  Source System           | Choose the one created earlier from the dropdown

Properties:

|  Name     | Value
|  :------------- | :-------------
|  **`Type`**           | **`HTTP`**
|  **`ProxyType`**           | **`Internet`**
|  **`URL`**          | The URL of your ABAP environment
|  **`Authentication`** | **`BasicAuthentication`**
|  **`User`**    | User name of ABAP instance communication user
|  **`Password`**   | Password of ABAP instance communication user
|  **`ips.date.variable.format`**   | **`yyyy-MM-dd`**

**Save** your changes.



### Run Identity Provisioning

After the source and target Systems have been created and connected with each other you can run the Identity provisioning.

Switch to **Source Systems**

![Navigate from Target System to Source Systems](IPS_target_system_2_source_systems.png)

Open your source system and select the **Jobs** tab.

Choose **Run Now**

![Run Identity Provisioning Job](IPS_source_system_run_job.png)

To check the status of the job run, select **Job Logs** from the navigation pane.

![Navigate to Job Logs](IPS_Job_logs_open.png)

Search for your log by checking the source system name and time and make sure the status is **Success**.

![Job finished successfully](IPS_Job_log_success.png)

>If the run did not finish successfully, you can navigate to the log and follow the instructions there to analyze and solve the problem. See also [Guided Answers: Identity Provisioning Troubleshooting](https://ga.support.sap.com/dtp/viewer/#/tree/2065/actions/26547:29111:29114:27412).



### Log On to ABAP Environment in Eclipse

Now that the Developer user has been provisioned and authorized in the ABAP environment for ABAP development, you can connect the user to the system by using ABAP Development Tools for Eclipse.

Open your Eclipse and navigate to **File > New > Project**.

![Start project creation in eclipse](ADT_project_new.png)

Choose **ABAP Cloud Project** and select **Next**

![Choose to create ABAP Cloud project](ADT_project_new_cloud.png)

Choose **SAP BTP ABAP Environment** > **Use a Service Key**  and select **Next**

![Choose to create from service key](ADT_project_new_service_key.png)

Paste the service key for Eclipse integration (see prerequisites)

![Paste Service Key for ADT usage](ADT_project_new_service_key_paste.png)

**Copy Logon URL to Clipboard**

![Copy Logon URL to Clipboard](ADT_project_new_open_logon.png)

Open an incognito browser window and paste the logon URL into the address line.
Choose your right identity provider URL.

![Identity Provider choice](ADT_project_new_logon_chose_idp.png)

Enter the credentials of the Developer User and log on.

![Log on with provisioned developer](ADT_project_new_logon_with_developer.png)

A success message is displayed and the browser window can be closed.

![Log on for ADT succeeded](ADT_project_new_logon_with_developer_success.png)

In the project wizard in Eclipse, check the ABAP environment and user data, that are displayed in the **Service Instance Connection** dialog and select **Finish**

![Finish ABAP Cloud project creation](ADT_project_new_finish.png)

The new project is displayed and you can start developing.

![See ABAP Cloud project in ADT navigation](ADT_project_created.png)




### Test yourself





---
