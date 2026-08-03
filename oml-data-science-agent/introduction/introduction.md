# Introduction

Estimated Workshop Time: 60 minutes


## About this workshop

The labs in this workshop walk you through all the steps to get started with Oracle Data Science Agent.

Data Science Agent is an intelligent built-in conversational chatbot integrated with Oracle Machine Learning UI included in your Oracle Autonomous AI Database subscription. You must provide the LLM, specified through your AI profile, whether from a third-party AI provider, OCI GenAI Service, or one you privately host. You can run complete data science workflows using natural language in the Data Science Agent chat.

As a human-in-the-loop chatbot, Data Science Agent requires your intervention and approval in the following areas:

* **Object association:** Associating database objects with a conversation improves precision and efficiency. Objects must be associated or approved before the agent creates views that use them; other supported operations may still use accessible objects based on the user's database privileges. 
* **Conversation management:** The scope of Data Science Agent is limited to the active conversation context only. You must create, manage, and delete your conversations. 
* **Profile selection:** You must select an AI profile, and therefore, the LLM that powers the agent. You can also switch profiles mid-conversation. 
* **Approval for object view:** During any conversation, Data Science Agent compares the object_list with the objects already associated with the active conversation. If required, the agent proposes any table or view outside the user-approved set. It stops the conversation and explicitly seeks your approval to associate those missing objects before continuing. If you approve associating the missing objects, the association is recorded and the agent continues the conversation. Else, the operation is canceled and no view is created.


Estimated Time: X

### Objectives

In this workshop, you will learn how to:

* Access and use the Oracle Machine Learning Data Science Agent
* Create AI Credential and AI Profile
* Create a Data Science Conversation
* Use the Data Science Agent chat interface
* Issue sequence of prompts to perform sequences of machine learning tasks


### Prerequisites

This workshop requires access to your Oracle Machine Learning UI in Oracle Autonomous AI Database. You may use your own cloud account or a training account whose details were given to you by an Oracle instructor.

* An Oracle Cloud Account with an Autonomous AI Database instance and Cloud Shell.
* Familiarity with OML UI
* Familiarity with Oracle Autonomous AI Database
* Familiarity with Oracle Cloud Infrastructure (OCI) GenAI Service
* Access to OCI GenAI Services LLM

> **Note:** If you have a **Free Trial** account, when your Free Trial expires your account will be converted to an **Always Free** account. You will not be able to conduct Free Tier workshops unless the Always Free environment is available. [Click here for the Free Tier FAQ page](https://www.oracle.com/cloud/free/faq.html).

## Acknowledgements

* **Author** - Moitreyee Hazarika, Consulting User Assistance Developer, Oracle AI Database User Assistance Development
* **Contributors** - Mark Hornick, Senior Director, Data Science and Machine Learning; Marcos Arancibia Coddou, Product Manager, Oracle Data Science; Sherry LaMonica, Consulting Member of Tech Staff, Machine Learning
* **Last Updated By/Date** - Moitreyee Hazarika, July 2026
