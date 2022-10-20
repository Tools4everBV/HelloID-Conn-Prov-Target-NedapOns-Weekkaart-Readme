# HelloID-Conn-Prov-Target-NedapONS-Weekkaart

| :information_source: Information |
|:---------------------------|
| This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.       |


> :warning: **_Information_**   <br />This connector only correlates a HelloID person with an employee account in Nedap Ons and updates the registration profile.
<br />Extensive knowledge of HelloID provisioning and Nedap Ons is required.

<br />
<br />

<p align="center">
  <img src="https://user-images.githubusercontent.com/68013812/94918899-c672c700-04b3-11eb-9132-7125bbf77fa5.png">
</p>

## Table of Contents

- [Introduction](#introduction)
- [Getting Started](#getting-started)
  * [Connection settings](#Connection-settings)
  * [Remarks](#Remarks)
  * [Provisioning](#provisioning)
  * [Employee Additional Mapping](#Employee-Additional-Mapping)
- [Setup the connector](#setup-the-connector)
- [HelloID Docs](#helloid-docs)

## Introduction
NedapONS is a healthcare application and provides a set of REST API's that allow you to programmatically interact with it's data.

## Getting Started

The _HelloID-Conn-Prov-Target-NedapONS-Weekkaart_ connector can be executed only on-premises using the HelloID-Agent, because of the mapping file and required certificate to authorize with Nedap.

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
- This connector only correlates a HelloID person with one or more employee accounts in NedapONS and updates the registration profile. It updates the registration profile of all the accounts from the contracts in the conditions.
- To empty a weekkaartprofiel use the text '<geen weekkaartprofiel>' as a placeholder.

### Provisioning

| Files       | Description                                             |
| ----------- | ------------------------------------------              |
| create.ps1  | Correlates a HelloID person with a NedapONS employee account and updates the registration profile |
| update.ps1  | The same script is used for both actions |
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
