# HelloID-Conn-Prov-Target-NedapONS-Weekkaart

> :warning: **_Information_**   <br />This connector only correlates a HelloID person with an employee account in NedapONS and updates the registration profile
<br />Extensive knowledge of HelloID provisioning and NedapONS is required.

<br />
<br />

<p align="center">
  <img src="https://user-images.githubusercontent.com/68013812/94918899-c672c700-04b3-11eb-9132-7125bbf77fa5.png">
</p>

## Table of Contents
- [Introduction](introduction)
- [Getting Started](getting-started)
  + [Connection settings](Connection-settings)
  + [Remarks](Remarks)
  + [Provisioning](provisioning)
  + [Employee Additional Mapping](Employee-Additional-Mapping)
- [Setup the connector](setup-the-connector)
- [HelloID Docs](helloid-docs)


## Introduction
NedapONS is a healthcare application and provides a set of REST API's that allow you to programmatically interact with it's data.

## Getting Started

The _HelloID-Conn-Prov-Target-NedapONS-Weekkaart_ connector is created for both Windows PowerShell 5.1 and PowerShell Core. This means that the connector can be executed in both cloud and on-premises using the HelloID agent.

### Connection settings

The following settings are required to connect to the API.

| Setting     | Description |
| ------------ | ----------- |
| Environment URL API     |    https://api-staging.ons.io                                     |
| Certificate (.PFX) Path    |  Fullpath to Certificate> Nedap-cert.pfx                       |
| Certificate Password |    Password of the certificate                                       |
| Mapping File |  The Path to the mapping file |
| CSV separation Character| Mapping File CSV Separation Character         |

### Remarks

- This connector only correlates a HelloID person with an employee account in NedapONS and updates the registration profile 

### Provisioning

| Files       | Description                                             |
| ----------- | ------------------------------------------              |
| create.ps1  | Correlates a HelloID person with an employee account in NedapONS and updates the registration profile |
| update.ps1  | updates the registration profile |

Using this connector you will have the ability to create and manage the following items in Nedap:

Create:
* Correlates a HelloID person with an employee account in NedapONS and updates the registration profile based on the contracts in condition from the Business Rules

Update:
* Updates the registration profile of the corresponding employee account as configured in the connector mapping.
----------

###  Employee Additional Mapping
| Header    | Description |
| ------------ | ----------- |
| Title.ExternalId   | Property of the HelloID primary contract
| Department.ExternalId  | Property of the HelloID primary contract
| EducationId   |   Deskundigheidsprofiel > Import Code
| RegistrationProfile | Weekkaartprofiel > ProfileName
| ClusterId  | Organigram > Identificatie
| ClusterName  |    Organigram > Naam

## HelloID Docs

The official HelloID documentation can be found at: https://docs.helloid.com/
