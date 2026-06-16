# ZavaShop Supplier Review Assistant

## About This Hack

ZavaShop is a fast-moving consumer goods retailer. The procurement team receives supplier quotes and contract extracts daily - each one needs to be manually reviewed against catalog benchmarks, supplier history, and commercial policy before a recommendation can be raised.

Today that process is slow, repetitive, and prone to missed details. Relevant information is spread across documents, spreadsheets, and internal records. A junior procurement analyst can spend 45 minutes on a single supplier review.

In this hack you will build an AI-assisted supplier review workflow. The assistant will extract structured commercial details from a sample supplier document, retrieve supporting context from prepared reference data, and generate a procurement review summary complete with recommendation, risk notes, and open questions - in seconds.

---

## What You Will Build

A two-phase AI assistant in Microsoft Foundry:

**Phase 1 - Document Intelligence:** An extraction agent that reads a raw supplier document, pulls out structured commercial fields (supplier name, quoted price, payment terms, delivery timeline, renewal conditions), and flags anything missing or ambiguous.

**Phase 2 - Grounded Review Assistant:** The same agent extended with an Azure AI Search knowledge base containing ZavaShop's product catalog benchmarks and supplier history. The assistant compares the extracted supplier data against that reference context and produces a procurement review output with a recommendation, supporting reasons, identified risks, and follow-up questions for the supplier.

---
## About the Sandbox Environment

   | Resources | Value | Remarks |
   | --- | --- | --- |
   | Enabled Services | `Microsoft Foundry` <br> `Azure OpenAI`<br> `Azure AI Search` <br> `Storage Accounts`  | You will have access to a dedicated, Owner role permissions on the subscription to explore any desired resources |
   | Azure Entra ID User | Pre-created Entra ID user account | You will get one Entra ID User Account. |
   | Azure Subscription Permissions | **Owner** privilege over Azure Subscription | You will get owner access to the Azure subscription. |
   | Azure Credit | **$50 USD**| Consumption limit is set on Azure spend to 50 USD. |
   | Credit Alerts | Credit Alerts are set on consumption of 25%, 50%, 75%, 85%, 90%, 95% and 100% of total Azure credits. |Make sure to check your registered email's inbox for any alert-related mails. Alerts give you a head start to keep your Azure spending in control and to plan out the remaining credits in the best way possible. |
   | Sandbox Duration | 1 Days/24 Hours or until Azure Consumption Credits are exhausted.  | The sandbox environment will be deleted automatically after 1 Days/24 Hours or once the Azure credits are exhausted, whichever comes first. |

## Support Contact

The CloudLabs support team is available 24/7, 365 days a year, via email and live chat to ensure seamless assistance at any time. We offer dedicated support channels tailored specifically for both learners and instructors, ensuring that all your needs are promptly and efficiently addressed.

Learner Support Contacts:

- Email Support: cloudlabs-support@spektrasystems.com  
- Live Chat Support: https://cloudlabs.ai/labs-support

Click **Next** at the bottom of the page to proceed to the next page.

   ![](./media/auto-it-gt-gr-g2.png)

## Happy Hacking!!
