# HelloID-Conn-Prov-Target-NedapOns-Weekkaart

> [!IMPORTANT]
> This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.

> [!NOTE]
> This connector only associates a HelloID person with their employee accounts in Nedap Ons and updates the Registration Profile (Weekkaart). Please refer to the [Fact Sheet](#fact-sheet) for more details.
>
> __Extensive knowledge of HelloID Provisioning and Nedap Ons (Nedap employee) is required.__


<p align="center">
  <img src="https://user-images.githubusercontent.com/68013812/94918899-c672c700-04b3-11eb-9132-7125bbf77fa5.png"
   alt="drawing" style="width:300px;"/>
</p>

> [!IMPORTANT] Upgrade Warning
> This V2 version of the connector has new names for the columns in the RegistrationProfile.csv mapping file. Make sure to change that accordingly in your RegistrationProfile.csv file when upgrading. A example can be found in the Asset folder.

> [!IMPORTANT] Upgrade warning!
> **Since the June 12, 2025 update, the way the Account Reference is stored has changed.**
> To upgrade from an existing V2 version to this version, should be performed in HelloID along with the "Update all Accounts" action to convert the Account Reference to the new format. If no force update is performed, reconciliation cannot match the existing account in HelloID with the one in Nedap. *See more: [Account Reference Conversion](#account-reference-conversion)*

## Table of contents

- [HelloID-Conn-Prov-Target-NedapOns-Weekkaart](#helloid-conn-prov-target-nedapons-weekkaart)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Supported features](#supported-features)
  - [Getting started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Connection settings](#connection-settings)
    - [Correlation configuration](#correlation-configuration)
    - [Available lifecycle actions](#available-lifecycle-actions)
    - [Field mapping](#field-mapping)
      - [The Primary Contract Calculation foreach employment](#the-primary-contract-calculation-foreach-employment)
      - [Script Mapping lookup values](#script-mapping-lookup-values)
  - [Remarks](#remarks)
    - [One Create/update script](#one-createupdate-script)
    - [Correlation](#correlation)
    - [Primary Contract](#primary-contract)
    - [IO Import validation](#io-import-validation)
    - [Preview Mode](#preview-mode)
    - [OutputInfo](#outputinfo)
    - [Always Update](#always-update)
    - [Account Reference Conversion](#account-reference-conversion)
  - [Mapping Remarks](#mapping-remarks)
    - [Example mappings](#example-mappings)
    - [Mapping Type](#mapping-type)
  - [Governance Remarks](#governance-remarks)
    - [Import](#import)
      - [Already linked accounts](#already-linked-accounts)
      - [EmployeeNumber](#employeenumber)
      - [Import configuration](#import-configuration)
      - [Account access](#account-access)
      - [Import Entitlement (Value Registration Profile)](#import-entitlement-value-registration-profile)
    - [Reconciliation](#reconciliation)
      - [Notification](#notification)
      - [Account loop](#account-loop)
  - [Employee Additional Mapping:](#employee-additional-mapping)
  - [Supported Properties](#supported-properties)
  - [Fact Sheet](#fact-sheet)
  - [Development resources](#development-resources)
    - [API endpoints](#api-endpoints)
    - [API documentation](#api-documentation)
  - [Getting help](#getting-help)
  - [HelloID docs](#helloid-docs)

## Introduction
This repository only contains the documentation. The source code can be found in a private repository and is only meant for internal use. Link to Repository: [Nedap Ons Weekkaart](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapONS-Weekkaart)

Nedap Ons provides an API and XML import method to programmatically interact with its services and data. This connector only correlates a HelloID person with one or more employee accounts in Nedap Ons and updates the `Registration Profile` (weekkaart). It updates the Registration Profiles of all the accounts from the contracts in the conditions.

## Supported features

The following features are available:

| Feature                                   | Supported | Actions                                                                     | Remarks |
| ----------------------------------------- | --------- | --------------------------------------------------------------------------- | ------- |
| **Account Lifecycle**                     | ✅         | Create, Update *(Updating Registration Profile only)*                                  |         |
| **Permissions**                           | ❌         | -                                                                           |         |
| **Resources**                             | ❌         | -                                                                           |         |
| **Entitlement Import: Accounts**          | ✅         | [Import remarks](#import)                                                   |         |
| **Entitlement Import: Permissions**       | ❌         | -                                                                           |         |
| **Governance Reconciliation Resolutions** | ✅         | Reconciliation *(Reporting Only)* [Governance Remarks](#governance-remarks) |         |


## Getting started

### Prerequisites
- **Nedap PFX Certificate** <br>
  A valid Nedap certificate (.PFX) *Tools4ever need to requests a certificate by Nedap to access the REST API*
- **Credentials IOImport**<br>
  Credentials for the IO Import *Different account credentials as the REST API*
- **Mapping Files**<br>
  Mapping between HR departments / functions and the registration profile (Weekkaart).

- **Custom HelloId Property**<br>
  A custom property on the HelloID Person contract with a combination of the employeeCode and EmploymentCode called __custom.NedapOnsIdentificationNo__
Example:
  ```javascript
  function getValue() {
      return sourceContract.PersonCode + "-" + sourceContract.EmploymentCode
  }
  getValue();
  ```

### Connection settings

The following settings are required to connect to the API.

| Setting                               | Description                                                                                                              | Mandatory |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------- |
| Environment (Rest)                    | The Nedap Environment of the API (Development, Staging, Production )                                                     | Yes       |
| Certificate (.PFX) Path               | <FullPath to Certificate> Nedap-Cert.pfx                                                                                 | Yes       |
| Certificate Password                  | Password of the certificate                                                                                              | Yes       |
| IO ImportUrl                          | The URL to the IOImport, Example: "https://<.ioservice.net/importws/import"                                              | Yes       |
| IO ImportUsername                     | The UserName to connect to the IO Import                                                                                 | Yes       |
| IO ImportPassword                     | The Password to connect to the IO Import                                                                                 | Yes       |
| CSV Mapping Registration Profile (Weekkaart) | The Path to the mapping file for Registration Profile (Weekkaart)                                                               | Yes       |
| CSV Separation Character              | Mapping File CSV Separation Character                                                                                    | Yes       |
| Encoding for the csv imports.         | Default value is 'Default'                                                                                               | Yes       |
| Import Only Active Employees.         | When toggled, The Import script will only import employees that have currently active contracts in the Nedap ONS system. | No        |
| Days before start of the contract.    | Days before start of the contract. Only used when the `Import Only Active Employees` is enabled.                         | No        |
| Days after end of the contract.       | Days after end of the contract. Only used when the `Import Only Active Employees` is enabled.                            | No        |


### Correlation configuration

The correlation configuration is used to specify which properties will be used to match an existing account within _NedapOns-Employee_ to a person in _HelloID_.

| Setting                   | Value                    |
| ------------------------- | ------------------------ |
| Enable correlation        | `True`                   |
| Person correlation field  | `ExternalId`             |
| Account correlation field | `_outputInfo.externalId` |

> [!IMPORTANT]
> Correlation should be enabled even when no direct person properties are used for correlation in the account Life Cycle. The correlation properties are solely used for the import and reconciliation. [See remark EmployeeNumber](#employeenumber).

> [!TIP]
> _For more information on correlation, please refer to our correlation [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems/correlation.html) pages_.

### Available lifecycle actions

The following lifecycle actions are available:

| Action             | Description                                                                                                             |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| create.ps1         | Updates the Registration Profile of the associated Nedap accounts based on the contracts in condition from the Business Rules. |
| configuration.json | Contains the connection settings and general configuration for the connector.                                           |
| fieldMapping.json  | Defines mappings between HelloID (person) fields and target system account fields.                                         |
| import.ps1         | Import the existing account from Nedap *(Configurable by active) *                                 |

### Field mapping

The field mapping can be imported using the `_fieldMapping.json` file.

In addition to the field mapping, some properties require extra mappings that are not included in the file. These properties are added directly in the code:
- `MutationAction`
- `Contract`
- `RegistrationProfile`


#### The Primary Contract Calculation foreach employment
The primary contract calculation is the same as the HelloID primary contract calculation. Note that the order of properties can be adjusted to meet the customer's requirements.

```Powershell
## Configuration
# Primary Contract Calculation foreach employment
$firstProperty =  @{ Expression = { $_.Details.Fte };             Descending = $true  }
$secondProperty = @{ Expression = { $_.Details.HoursPerWeek };    Descending = $true  }
$thirdProperty =  @{ Expression = { $_.Details.Sequence };        Descending = $true  }
$fourthProperty = @{ Expression = { $_.EndDate };                 Descending = $true  }
$fifthProperty =  @{ Expression = { $_.StartDate };               Descending = $false }
$sixthProperty =  @{ Expression = { $_.ExternalId };              Descending = $false }

# Priority Calculation Order (High priority -> Low priority)
$splatSortObject = @{
    Property = @(
        $firstProperty,
        $secondProperty,
        $thirdProperty,
        $fourthProperty,
        $fifthProperty,
        $sixthProperty)
}
```

#### Script Mapping lookup values
```Powershell
# Lookup value which is used in the mapping to determine the Registration profile (Weekkaart)
$eduRegTeamPrimaryLookupKey    = { $_.Title.ExternalId }      # Mandatory
$eduRegTeamSecondaryLookupKey  = { $_.Department.ExternalId } # Not Mandatory
```

## Remarks
### One Create/update script
The script used for updating is identical to the script used for creating.

### Correlation

Because Nedap contains multiple accounts per HelloID person (contract), accounts are correlated based on a custom contract property called `custom.NedapOnsIdentificationNo`. An example of this can be found in the [Prerequisites](#prerequisites) section. This approach is necessary because HelloID does not support correlation based on contract properties.

> [!TIP] The import and reconciliation functionality uses the correlation directly on the Person property. [See remark Import](#import)

### Primary Contract
Since the connector supports multiple accounts for a single HelloID person (per contract), the default primary contract calculation may not always be applicable. The connector includes an example of a primary contract calculation that determines the primary contract for each employment in Nedap.

### IO Import validation
When updating an employee account, it's not possible to directly verify whether the _Weekkaart_ has been successfully updated in Nedap. To confirm this, you need to check the_ Import Report_ in the Nedap UI by going to: _Beheer > Import > Importrapportage inzien._

### Preview Mode
Note that in preview mode (DryRun), all HelloID contracts of a Person are in scope. Therefore, it does not simulate the actual outcome when it comes to determining which account should be created, updated, or deleted. However, this DryRun mode is added to verify if the mapping, configuration settings, etc. are present and correct. The contracts in scope are normally configured in the business rules. This cannot be stimulated in Preview.

### OutputInfo
In the field mapping, some properties have a `_outputInfo` prefix. These properties are **only** used to return data from the connector, and the information is visible in the account data or in the notification. Since there can be multiple accounts, the connector returns the values as comma-separated strings. Note that there is **no link** between the individual values.
- Registration Profile
- NedapOnsIdentificationNo
- externalId *(Only used in Import.ps1)*

### Always Update
The Update action always updates because the current Registration Profile (Weekkaart) on the Nedap account cannot be retrieved.

### Account Reference Conversion
To support Nedap Account import scripts, the account reference object has been changed to a HashTable. Previously, the account reference was stored as an array. If you already have a Nedap Weekkaart V2 version running, you can click "Update all Accounts" and run an Enforcement. This will convert the "old" account references to the current format.

## Mapping Remarks
### Example mappings
Example mappings can be found in the Assets folder.

### Mapping Type
The connector is built containing a single property which cannot be mapped directly from the person model in HelloID. So there is a mapping file required. This file must include a mapping between HR departments and/or function to a Nedap Registration profile. The actions of the connector fails, when there is no mapping found.

## Governance Remarks
The Nedap connector supports importing Nedap employee accounts, and the import functionality can be used as normal. However, there are some important remarks to keep in mind, see the  [Import](#import) remarks section for details. Reconciliation is intended for **reporting** purposes only.

### Import
#### Already linked accounts
The import only detects users without an existing linked Nedap account in HelloID. Users who already have a linked account and receive additional Nedap accounts won’t appear in the entitlement import. Therefore, the import script is primarily intended for initial implementation.

#### EmployeeNumber
To perform person correlation, the IdentificationNo is split on the dash (-) to extract the employee number. This differs from the approach used in the Account Lifecycle, where a custom property combining the employee number and contract number is used to correlate individual accounts.

To store the employee number, the `_outputInfo.externalId` property is used in the field mapping. Please note that this field is only used by the import script.

#### Import configuration
The import scripts include configuration options for which accounts to retrieve. By default, all accounts are retrieved and shown in the import overview, but you may want to exclude past accounts. This can be done by toggling `ImportOnlyActiveEmployees`, which retrieves only accounts with active contracts.

The `ImportOnlyActiveEmployees` toggle also enables additional settings to expand the range of active contracts considered, using `daysBeforeContractStartDate` and `daysAfterContractEndDate`. These values can be adjusted in the configuration.

#### Account access
Account access is not used in the Nedap Employee Connector; therefore, no `Account Access` entitlements will be granted.

#### Import Entitlement (Value Registration Profile)
The Nedap API does not support retrieving the Registration Profile value, this value cannot be imported directly. However, this is handled automatically during the update process, which is triggered after accounts are correlated by the import actions.

### Reconciliation
Reconciliation for Nedap cannot be fully used because there is a one-to-many relationship between a HelloID person and Nedap accounts. Due to the risk of unwanted account deletions, reconciliation should be used only as a reporting tool to compare HelloID against Nedap. The deletion action is not used, but creating accounts is still possible—although this is not yet fully supported.

#### Notification
Reconciliation does not support custom or built-in events when deleting accounts through reconciliation.

#### Account loop
Due to multiple accounts per HelloID person, old (unmanaged) accounts cause managed persons in HelloID to repeatedly appear in each report because of mismatches between accounts managed by HelloID and those still existing in Nedap. You cannot exclude the person entirely since they are still in scope, but only some of their accounts are not. Additionally, excluding accounts at this level is not possible.

To minimize this issue, you are able to filter in the import script to retrieve only active accounts. See [Import configuration](#import-configuration)

## Employee Additional Mapping:
| Header                       | Description                                             |
| ---------------------------- | ------------------------------------------------------- |
| Primary Contract calculation | Example of a Primary contract calculation.              |
| HelloIDPrimaryLookupKey      | The Primary property of the HelloID primary contract per employment   |
| HelloIDSecondaryLookupKey    | The Secondary Property of the HelloID primary contract per employment |
| RegistrationProfile          | Weekkaartprofiel > ProfileName                          |

> [!TIP]
> That the mapped value will be created if they do not exist in Nedap! So if you choose a RegistrationProfile that does not exist, it will be created. What might encounter some unexpected behavior.*

## Supported Properties

| PropertyName        | Notes                           |
| ------------------- | ------------------------------- |
| Id                  | IdentificationNo                |
| RegistrationProfile | Weekkaartprofiel => ProfileName |

## Fact Sheet
The following table displays an overview of the functionality for the Nedap Ons connector for HelloID Provisioning and Service Automation.

| Type of action                       | Nedap | HelloID provisioning                                                         | HelloID Service Automation |
| ------------------------------------ | ----- | ---------------------------------------------------------------------------- | -------------------------- |
| Create Employees                     | Yes   | No, *Only the registration Profile (Weekkaart) will be set.*  no scheduling* | No                         |
| Update Employees                     | Yes   | No, *Only the registration Profile (Weekkaart) will be set.*  no scheduling* | No                         |
| Set registration Profile (Weekkaart) | Yes   | Yes, *Additional mapping required*                                           | No                         |


## Development resources

### API endpoints

The following endpoints are used by the connector

| Endpoint                                               | Description                        | Type     |
| ------------------------------------------------------ | ---------------------------------- | -------- |
| /v0/administration/employees/by_identification_no/<id> | Retrieve Employee Information      | Rest     |
| /importws/import                                       | CRUD Employee Information          | IOImport |
| /v0/xstream/employees/data                             | Retrieve Employee Information List | Rest     |
| /v0/xstream/contracts/data                             | Retrieve Contract Information      | IOImport |

### API documentation

* Nedap API documentation → [klik](https://www.ons-api.nl/english/technical/APIS.html)
* Nedap XML IOImport Manual → [klik](https://support.nedap-ons.nl/support/solutions/articles/103000265750-gegevens-importeren-via-xml-bestanden)
* Nedap XML IOImport Documentation → [klik](https://hc-freshdesk-assets.s3.eu-central-1.amazonaws.com/Supportportaal%20Ons%20Suite/Technische%20documenten/XML%20Interface/io_server_xml_interface.pdf)
* Nedap ONS authorization Manual → [klik](https://www.ons-api.nl/english/authorization/AuthorizationInOns.html)

## Getting help

> [!TIP]
> _For more information on how to configure a HelloID PowerShell connector, please refer to our [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems.html) pages_.

> [!TIP]
>  _If you need help, feel free to ask questions on our [forum](https://forum.helloid.com/forum/helloid-connectors/provisioning/312-helloid-prov-target-nedap-ons-employee)_.

## HelloID docs

The official HelloID documentation can be found at: https://docs.helloid.com/