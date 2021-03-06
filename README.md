# Collection of Azure Sentinel - Analytics Rules (Template)

> Austin Lai | April 30th, 2022

---

<!-- Description -->

A collection of Azure Sentinel - Analytics Rules (Template) for your reference.

<!-- /Description -->

## Table of Contents

<!-- TOC -->

- [Collection of Azure Sentinel - Analytics Rules (Template)](#collection-of-azure-sentinel---analytics-rules-template)
    - [Table of Contents](#table-of-contents)
    - [Analytics Rules (Template)](#analytics-rules-template)

<!-- /TOC -->

## Analytics Rules (Template)

<details><summary>Analytics Rules (Template)</summary>

```json
//  Replace "XXXXXX" to your own Subscription ID and Resource Group accordingly
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspace": {
            "type": "String"
        }
    },
    "resources": [
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/bd86f346-1009-470d-94cb-309004eb2bf7')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/bd86f346-1009-470d-94cb-309004eb2bf7')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "NRT",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - NRT - SSH - Potential Brute Force",
                "description": "",
                "severity": "High",
                "enabled": true,
                "query": "let threshold = 5;\r\nSyslog\r\n| where (SyslogMessage contains \"Failed password for invalid user\" or SyslogMessage contains \"invalid user\" or SyslogMessage contains \"Failed password\") \r\n| where ProcessName =~ \"sshd\" \r\n//| parse kind=relaxed SyslogMessage with * \"invalid user\" user \" from \" ip \" port\" port \" ssh2\"\r\n| extend\r\n    user = extract(@\"(?:^Failed password for invalid user |^Failed password for |^Invalid user )(\\S+)\", 1, SyslogMessage),\r\n    ip = extract(\"(([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.(([0-9]{1,3})))\", 1, SyslogMessage),\r\n    port = extract(@\".*?port\\s(\\S+)\", 1, SyslogMessage)\r\n| project user, ip, port, SyslogMessage, EventTime, Computer\r\n| summarize EventTimes = make_list(EventTime), PerHourCount = count() by ip, bin(EventTime, 4h), user, Computer\r\n| where PerHourCount > threshold\r\n| mvexpand EventTimes\r\n| extend EventTimes = tostring(EventTimes) \r\n| summarize\r\n    StartTimeUtc = min(EventTimes),\r\n    EndTimeUtc = max(EventTimes),\r\n    UserList = makeset(user),\r\n    sum(PerHourCount)\r\n    by IPAddress = ip, Computer\r\n| extend UserList = tostring(UserList) \r\n| extend\r\n    timestamp = StartTimeUtc,\r\n    IPCustomEntity = IPAddress,\r\n    AccountCustomEntity = UserList",
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "CredentialAccess"
                ],
                "techniques": [
                    "T1110"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": true,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": [
                    {
                        "entityType": "Account",
                        "fieldMappings": [
                            {
                                "identifier": "FullName",
                                "columnName": "AccountCustomEntity"
                            }
                        ]
                    },
                    {
                        "entityType": "IP",
                        "fieldMappings": [
                            {
                                "identifier": "Address",
                                "columnName": "IPCustomEntity"
                            }
                        ]
                    },
                    {
                        "entityType": "Host",
                        "fieldMappings": [
                            {
                                "identifier": "HostName",
                                "columnName": "Computer"
                            }
                        ]
                    }
                ],
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/c083cd39-d982-4c75-8335-c849ed133b9c')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/c083cd39-d982-4c75-8335-c849ed133b9c')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "NRT",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - NRT - Detect changes in Network interfaces should not have public IPs policy",
                "description": "",
                "severity": "Medium",
                "enabled": true,
                "query": "AzureActivity\n| where parse_json(Properties).entity contains \"policyDefinitions/XXXXXX\" or parse_json(Properties).entity contains \"policyassignments/XXXXXX\"\n| where isnotempty(ActivityStatusValue)\n    and isnotnull(Properties_d) == true\n    and isnotnull(parse_json(Properties_d).requestbody)\n| sort by TimeGenerated desc \n\n",
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [],
                "techniques": [],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": true,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [
                            "Account"
                        ],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "alertDetailsOverride": null,
                "customDetails": {
                    "OperationNameValue": "OperationNameValue",
                    "Properties_d": "Properties_d"
                },
                "entityMappings": [
                    {
                        "entityType": "Account",
                        "fieldMappings": [
                            {
                                "identifier": "FullName",
                                "columnName": "Caller"
                            }
                        ]
                    }
                ],
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/b861b58f-542e-4762-be4a-13011d49843f')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/b861b58f-542e-4762-be4a-13011d49843f')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - SSH - Potential Brute Force",
                "description": "Identifies an IP address that had 5 failed attempts to sign in via SSH in a 4 hour block during a 24 hour time period.",
                "severity": "High",
                "enabled": true,
                "query": "let threshold = 5;\nSyslog\n| where (SyslogMessage contains \"Failed password for invalid user\" or SyslogMessage contains \"invalid user\" or SyslogMessage contains \"Failed password\") \n| where ProcessName =~ \"sshd\" \n//| parse kind=relaxed SyslogMessage with * \"invalid user\" user \" from \" ip \" port\" port \" ssh2\"\n| extend\n    user = extract(@\"(?:^Failed password for invalid user |^Failed password for |^Invalid user )(\\S+)\", 1, SyslogMessage),\n    ip = extract(\"(([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.(([0-9]{1,3})))\", 1, SyslogMessage),\n    port = extract(@\".*?port\\s(\\S+)\", 1, SyslogMessage)\n| project user, ip, port, SyslogMessage, EventTime, Computer\n| summarize EventTimes = make_list(EventTime), PerHourCount = count() by ip, bin(EventTime, 4h), user, Computer\n| where PerHourCount > threshold\n| mvexpand EventTimes\n| extend EventTimes = tostring(EventTimes) \n| summarize\n    StartTimeUtc = min(EventTimes),\n    EndTimeUtc = max(EventTimes),\n    UserList = makeset(user),\n    sum(PerHourCount)\n    by IPAddress = ip, Computer\n| extend UserList = tostring(UserList) \n| extend\n    timestamp = StartTimeUtc,\n    IPCustomEntity = IPAddress,\n    AccountCustomEntity = UserList\n\n",
                "queryFrequency": "P1D",
                "queryPeriod": "P1D",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "CredentialAccess"
                ],
                "techniques": [
                    "T1110"
                ],
                "alertRuleTemplateName": "e1ce0eab-10d1-4aae-863f-9a383345ba88",
                "incidentConfiguration": {
                    "createIncident": true,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": [
                    {
                        "entityType": "IP",
                        "fieldMappings": [
                            {
                                "identifier": "Address",
                                "columnName": "IPCustomEntity"
                            }
                        ]
                    },
                    {
                        "entityType": "Host",
                        "fieldMappings": [
                            {
                                "identifier": "HostName",
                                "columnName": "Computer"
                            }
                        ]
                    },
                    {
                        "entityType": "Account",
                        "fieldMappings": [
                            {
                                "identifier": "Name",
                                "columnName": "UserList"
                            }
                        ]
                    }
                ],
                "sentinelEntitiesMappings": null,
                "templateVersion": "1.0.0"
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/9e720708-2ba6-47e9-8b93-9978198517d2')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/9e720708-2ba6-47e9-8b93-9978198517d2')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Successful SSH Brute Force Attack VM Detected",
                "description": "",
                "severity": "High",
                "enabled": true,
                "query": "Syslog\r\n| where ProcessName =~ \"sshd\" \r\n| where SyslogMessage contains \"Accepted publickey\" or SyslogMessage contains \"Accepted password\"\r\n| extend\r\n    user = extract(@\"(?:^Accepted publickey for |^Accepted password for )(\\S+)\", 1, SyslogMessage),\r\n    ip = extract(\"(([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.(([0-9]{1,3})))\", 1, SyslogMessage),\r\n    port = extract(@\".*?port\\s(\\S+)\", 1, SyslogMessage)\r\n| where user in ( \r\n    ( _GetWatchlist('SSH-Brute-Force-user-list')\r\n    | project SSHBruteForceUserList))",
                "queryFrequency": "PT5H",
                "queryPeriod": "PT5H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "CredentialAccess",
                    "InitialAccess"
                ],
                "techniques": [
                    "T1110",
                    "T1133",
                    "T1078"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": true,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": [
                    {
                        "entityType": "Account",
                        "fieldMappings": [
                            {
                                "identifier": "FullName",
                                "columnName": "user"
                            }
                        ]
                    },
                    {
                        "entityType": "IP",
                        "fieldMappings": [
                            {
                                "identifier": "Address",
                                "columnName": "ip"
                            }
                        ]
                    },
                    {
                        "entityType": "Host",
                        "fieldMappings": [
                            {
                                "identifier": "HostName",
                                "columnName": "Computer"
                            }
                        ]
                    }
                ],
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/03b2f6f6-3242-4e90-84ee-17b5aa2dcc17')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/03b2f6f6-3242-4e90-84ee-17b5aa2dcc17')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux add user to group via groupadd",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n | where ProcessName == \"groupadd\" and ( SyslogMessage !contains \"syslog\" or  SyslogMessage !contains \"omsagent\" )\r\n",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "DefenseEvasion",
                    "Persistence",
                    "PrivilegeEscalation"
                ],
                "techniques": [
                    "T1078",
                    "T1098",
                    "T1078",
                    "T1078"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/5b010145-9b13-4676-8af3-e5f58c51b945')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/5b010145-9b13-4676-8af3-e5f58c51b945')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux add user via useradd",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n | where ProcessName == \"useradd\" and ( SyslogMessage !contains \"syslog\" or  SyslogMessage !contains \"omsagent\" )\r\n",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "DefenseEvasion",
                    "Persistence",
                    "PrivilegeEscalation"
                ],
                "techniques": [
                    "T1078",
                    "T1136",
                    "T1078",
                    "T1078"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/45ff6562-365b-4153-85ce-28c908f1fede')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/45ff6562-365b-4153-85ce-28c908f1fede')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux delete user from system via userdel",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n| where ProcessName == \"userdel\"",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "Impact"
                ],
                "techniques": [
                    "T1531"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/f5bdee7e-292d-438b-b13b-f5fffd713a3b')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/f5bdee7e-292d-438b-b13b-f5fffd713a3b')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux monitor sudo command",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n | where ProcessName == \"sudo\" and SyslogMessage !contains \"omsagent\" and SyslogMessage !contains \"session opened\" and SyslogMessage !contains \"session closed\" and SyslogMessage !contains \"waagent\"\r\n//  | summarize by SyslogMessage\r\n//  | parse kind=relaxed SyslogMessage with * \"\"\r\n",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [],
                "techniques": [],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/647ec917-3d9d-4b4b-80e5-42027cafa0b6')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/647ec917-3d9d-4b4b-80e5-42027cafa0b6')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux Password Change via passwd",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n| where ProcessName == \"passwd\"\r\n| parse kind=relaxed SyslogMessage with * \"password changed for \" USER\r\n",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "Impact",
                    "Persistence"
                ],
                "techniques": [
                    "T1531",
                    "T1098"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/5c493fe4-a430-4dfe-aa7c-c3c63a4c4cdc')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/5c493fe4-a430-4dfe-aa7c-c3c63a4c4cdc')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux SSH Root Failed Attempt",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n| where (SyslogMessage contains \"invalid user\" or SyslogMessage contains \"Failed password\") \r\n| where SyslogMessage !contains \"Disconnected\" and SyslogMessage !contains \"Connection closed\"\r\n| where ProcessName =~ \"sshd\"\r\n| extend\r\n    USER = extract(@\"(?:^Failed password for invalid user |^Failed password for |^Invalid user )(\\S+)\", 1, SyslogMessage),\r\n    S_IPaddr = extract(\"(([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.([0-9]{1,3})\\\\.(([0-9]{1,3})))\", 1, SyslogMessage),\r\n    S_Port = extract(@\".*?port\\s(\\S+)\", 1, SyslogMessage)\r\n| where USER == \"root\"\r\n| summarize Count = count() by S_IPaddr, USER, Computer,_ResourceId",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "CredentialAccess"
                ],
                "techniques": [
                    "T1110"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": false,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/868ef531-e370-4b5e-92a9-be79123cc2ce')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/868ef531-e370-4b5e-92a9-be79123cc2ce')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux SSH with publickey",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n | where SyslogMessage contains \"Accepted publickey\"\r\n | where ProcessName =~ \"sshd\"\r\n | parse kind=relaxed SyslogMessage with * \"Accepted publickey for \" USER \" from \" S_IPaddr \" port\" S_Port \" ssh2\" *\r\n",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "InitialAccess"
                ],
                "techniques": [
                    "T1133",
                    "T1078"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/c85cb96d-889e-43bd-a6ca-24ad5d06ce08')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/c85cb96d-889e-43bd-a6ca-24ad5d06ce08')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules",
            "kind": "Scheduled",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "displayName": "Customized - Linux switch user to root via su command",
                "description": "",
                "severity": "Low",
                "enabled": true,
                "query": "Syslog\r\n | where ProcessName == \"su\" and SyslogMessage !contains \"omsagent\"\r\n | parse kind=relaxed SyslogMessage with * \"su for \" USER \" by \" *",
                "queryFrequency": "PT1H",
                "queryPeriod": "PT1H",
                "triggerOperator": "GreaterThan",
                "triggerThreshold": 0,
                "suppressionDuration": "PT5H",
                "suppressionEnabled": false,
                "tactics": [
                    "PrivilegeEscalation"
                ],
                "techniques": [
                    "T1078"
                ],
                "alertRuleTemplateName": null,
                "incidentConfiguration": {
                    "createIncident": false,
                    "groupingConfiguration": {
                        "enabled": true,
                        "reopenClosedIncident": false,
                        "lookbackDuration": "PT5H",
                        "matchingMethod": "AllEntities",
                        "groupByEntities": [],
                        "groupByAlertDetails": [],
                        "groupByCustomDetails": []
                    }
                },
                "eventGroupingSettings": {
                    "aggregationKind": "SingleAlert"
                },
                "alertDetailsOverride": null,
                "customDetails": null,
                "entityMappings": null,
                "sentinelEntitiesMappings": null
            }
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/bd86f346-1009-470d-94cb-309004eb2bf7/actions/e25549f98b75491699205c6d0c4c9230')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/bd86f346-1009-470d-94cb-309004eb2bf7/e25549f98b75491699205c6d0c4c9230')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules/actions",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "ruleId": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/bd86f346-1009-470d-94cb-309004eb2bf7')]",
                "triggerUri": "[listCallbackURL(concat('/subscriptions/XXXXXX/resourceGroups/XXXXXX/providers/Microsoft.Logic/workflows/test','/triggers/Microsoft_Sentinel_alert'),'2016-06-01').value]",
                "logicAppResourceId": "/subscriptions/XXXXXX/resourceGroups/XXXXXX/providers/Microsoft.Logic/workflows/test",
                "operatesOn": "Alert"
            },
            "dependsOn": [
                "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/bd86f346-1009-470d-94cb-309004eb2bf7')]"
            ]
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/c083cd39-d982-4c75-8335-c849ed133b9c/actions/69c71f5d291a4362a2de84dbc96c0888')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/c083cd39-d982-4c75-8335-c849ed133b9c/69c71f5d291a4362a2de84dbc96c0888')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules/actions",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "ruleId": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/c083cd39-d982-4c75-8335-c849ed133b9c')]",
                "triggerUri": "[listCallbackURL(concat('/subscriptions/XXXXXX/resourceGroups/XXXXXX/providers/Microsoft.Logic/workflows/Alert-changes_detected_Network_interfaces_should_not_have_public_IPs_policy','/triggers/Microsoft_Sentinel_alert'),'2016-06-01').value]",
                "logicAppResourceId": "/subscriptions/XXXXXX/resourceGroups/XXXXXX/providers/Microsoft.Logic/workflows/Alert-changes_detected_Network_interfaces_should_not_have_public_IPs_policy",
                "operatesOn": "Alert"
            },
            "dependsOn": [
                "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/c083cd39-d982-4c75-8335-c849ed133b9c')]"
            ]
        },
        {
            "id": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/b861b58f-542e-4762-be4a-13011d49843f/actions/e25549f98b75491699205c6d0c4c9230')]",
            "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/b861b58f-542e-4762-be4a-13011d49843f/e25549f98b75491699205c6d0c4c9230')]",
            "type": "Microsoft.OperationalInsights/workspaces/providers/alertRules/actions",
            "apiVersion": "2021-10-01-preview",
            "properties": {
                "ruleId": "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/b861b58f-542e-4762-be4a-13011d49843f')]",
                "triggerUri": "[listCallbackURL(concat('/subscriptions/XXXXXX/resourceGroups/XXXXXX/providers/Microsoft.Logic/workflows/test','/triggers/Microsoft_Sentinel_alert'),'2016-06-01').value]",
                "logicAppResourceId": "/subscriptions/XXXXXX/resourceGroups/XXXXXX/providers/Microsoft.Logic/workflows/test",
                "operatesOn": "Alert"
            },
            "dependsOn": [
                "[concat(resourceId('Microsoft.OperationalInsights/workspaces/providers', parameters('workspace'), 'Microsoft.SecurityInsights'),'/alertRules/b861b58f-542e-4762-be4a-13011d49843f')]"
            ]
        }
    ]
}
```

</details>

[Link to the file of Analytics Rules (Template) here](https://github.com/austin-lai/Collection-of-AzureSentinel-AnalyticsRules-Template/blob/master/Analytics%20Rules%20(Template).json)

<br />

---

> Do let me know any command or step can be improve or you have any question you can contact me via THM message or write down comment below or via FB
