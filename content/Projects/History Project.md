---
status: Completed
technical_owner: Sarah Choong
strategic_owner: Sarah Choong
repository: https://github.com/cpellet/history-project
stack: Python, LLM (Claude Haiku), Langchain
---
**Background:** The History Project at OCHA aims to gather and make publicly available all data about interagency humanitarian response plans across all years since UNDRO (see [the History Project report](https://unitednations.sharepoint.com/:w:/r/sites/OCHAAPMBMonitoringandTools/Shared%20Documents/HPC%20Tools%20and%20Field%20support/HISTORY%20PROJECT/Explanations/REPORT%20on%20the%20History%20Project%20oct%2025.docx?d=wafed53219b6f4ae48be6c172ab1001d6&csf=1&web=1&e=FQLjSO)). Accomplishing this feat requires gathering all interagency response plans from the 1990s onwards, digitizing their contents, and extracting useful information for further analysis. The latter step involves defining the scope of the information to extract, and setting up appropriate LLM-guided pipelines to produce a reliable and responsible dataset (see [[Providing data responsibly in the age of AI|this commentary]]).
## Preliminary work
Over several months, significant work was undertaken by the Monitoring team to discover, collect and scan past documents from OCHA and its predecessors. This process was documented at length in [David Goetghebuer's report](https://unitednations.sharepoint.com/:w:/s/OCHAAPMBMonitoringandTools/IQAhU-2vb5vkSovmwXKrEAHWAX-2biOG4oD0-b4atL6c3b0?e=2Urhpj), the [October 2025 presentation](https://unitednations.sharepoint.com/:p:/s/OCHAAPMBMonitoringandTools/IQB8og8BnAbxRIn-dg9QOQFYAduU8hug7ZCxDwMblL6ufs8?e=d0jbEM) and [Sanifa Shirinova's handover notes](https://unitednations.sharepoint.com/:w:/s/OCHAAPMBMonitoringandTools/IQDjDjgc-5wzRa-f2OaHtzvUAQhkIQVfb9btcwsmNk8C7AA?e=e1QhEt).

The result of this incredible work consists (amongst other artifacts) of a [folder](https://unitednations.sharepoint.com/:f:/r/sites/OCHAAPMBMonitoringandTools/Shared%20Documents/HPC%20Tools%20and%20Field%20support/HISTORY%20PROJECT/All%20Plans?csf=1&web=1&e=ug8hwk) containing all scanned inter-agency plans from 1992 to 2026, classified by year and crisis.
## Phase 2: Extraction
Work on the second phase of the project took place over a day, during which David's daughter Barbara joined us for her *stage de 3ème*. The task of extracting information from the digitized plans was broken down into three distinct steps:
1) Defining a **common schema** spanning plans to decide which key pieces of data to extract
2) Designing an **LLM-assisted pipeline** with humans-in-the-loop to perform the extraction
3) Performing an **initial analysis** of the data on the resulting dataset

Due to severe cost-related constraints for innovation projects at OCHA, the majority of the work was done using personal resources on a commercially-available AI engineering platform.
### 1. Common schema definition
A `Plan` entity was defined in the platform to establish a common schema between the documents. Chosen properties extracted from plans needed to satisfy a set of criteria in order to result in a high-quality dataset:
- Existing across the years in question (some concepts such as People Prioritized did not exist until recently and therefore could not be expected to be extracted reliably for all plans)
- Comparable across plans (financial figures needed to be adjusted for inflation and properly converted across currencies to ensure they could be compared)
- Simple to describe (data points such as financial requirements per cluster were too flaky to reliably extract using LLMs as they required many domain-specific insights)
- Easy to standardize (Consolidated Appeal, Appeal, Regional Appeal and Emergency Appeal all refer to the same concept, and should be treated as one)
- Self-defined (numerical figures needed to be specified in clear within documents, and not implicitly calculated from several pieces of information)

![[Pasted image 20260305154405.png|The Plan entity definition|326]] ![[Pasted image 20260305154502.png|Some properties assigned to plans|342]]

The resulting property set was defined as followed:
- The `Official Title` of the plan (e.g: *Drought Emergency in Southern Africa (DESA) Situation Report*)
- The `Appeal Type` (e.g: *Situation Report*)
- The `Cluster Names` mentioned in the plan (e.g: *Food Security, Health, WASH, etc...*)
- The `Currency` of monetary figures mentioned in the plan (e.g: *USD*)
- The `Geographic Scope` of the plan (*Country/Regional/Global*)
- `Participating Agencies` mentioned in the plan (e.g: *WFP, UNICEF, FAO, UNHCR, etc...*)
- The `Primary Crisis Type` of the plan (e.g: *Drought*)
- The `Publication Date` of the document (e.g: *Aug 1, 1992*)
- The `Specific Crisis Name` of the plan (e.g: *Drought Emergency in Southern Africa (DESA)*)
- The `Target Countries` of the plan (e.g: *Angola, Botswana, Lesotho, Malawi, etc...*)
- The `Total Financial Requirements` of the plan (e.g: *854000000*) (see `Currency`)
- The `Total People in Need` identified in the plan (e.g: *18000000*)
- The *Total People Targeted* by the plan (e.g: *0*) (0 if not identified)
- The `Validity End Date` of the appeal (e.g: *Dec 31, 1992*)
- The `Validity Start Date` of the appeal (e.g: *Aug 1, 1992*) (`Publication Date` if not mentioned)

These property definitions included a type (String/Integer/Date/List of Strings), a description, and a list of possible values where applicable (see `Geographic Scope`). This, along with instructions in the System Prompt (see below) ensured consistency across documents.
### 2. LLM-assisted pipeline
With the extraction schema defined, an LLM-assisted pipeline could be engineered within the platform to extract data and create a `Plan` entity for each document:

![[Pasted image 20260305161518.png|The pipeline feeds the OCR text into an LLM and uses its reply to construct `Plan` entities]]

An industry-standard model (*Claude Haiku 4.5* from *Anthropic*) was used in single completion mode with a carefully-crafted system prompt:

```markdown
You are an expert Humanitarian Data Analyst specializing in parsing historical and modern UN documents. Your goal is to extract structured metadata from a Humanitarian Needs and Response Plan (HNRP) or Appeal.

The input may be a pristine modern PDF or a noisy OCR scan from the 1980s/90s. You must be resilient to OCR errors (e.g., treating "l992" as "1992").

## EXTRACTION RULES

**1. Title & Subtitle:**

* **Official Title:** Extract the main header from the cover page (e.g., "Humanitarian Priorities Plan", "Consolidated UN-SADC Appeal").

* **Subtitle/Type:** Look for modifiers like "Midterm Review", "Flash Appeal". Do not include dates, only generic types.

**2. Geographic Scope:**

* Determine if this plan covers a single country or a region. Possible values: country, regional, global.

* **Target Countries:** If it is a Regional plan (like the 1992 Southern Africa example), list ALL specific countries listed on the cover or in the "Country Specific" section. If it is a single country plan, list that country.

**3. Crisis Details:**

* **Primary Crisis Type:** Identify the main driver by choosing one of the following: Drought, Flood, Energy Crisis, Cyclone, Conflict, Earthquake, Landslide, Tsunami, Economic Crisis, Tornado, Volcano or Disease Outbreak. If multiple crises match, return Complex Emergency.

* **Specific Name:** If the crisis has a name (e.g., "Cyclone Ditwah" or "2018-2019 Pakistan Drought"), extract it. If not, return "unknown". Where it makes sense, include the year (e.g: "2003 Cholera Outbreak").

* **Appeal Type:** Identify the main appeal type of the document by choosing one of the following: Flash Appeal, GHO, HNRP, Fact Sheet, Mid-Year Update or Addendum. If you are sure the document doesn't match any of those, create another one (such as "Consolidated Appeal (CAP)"), but do not include any dates.

* **Clusters:**Identify a list of clusters mentioned in the document and choose amongst the following: Education, Food Security, Health, Protection, Shelter/ CCCM/ NFI, WASH, Early Recovery, Nutrition, Multipurpose Cash Assistance, Gender-based Violence, Telecommunication, Logistics or Child Protection. Do not create new clusters, choose from one of the above verbatim!

**4. Temporal Data:**

* **Publication Date:** Look for "Issued on," "December 1992," or dates near the footer of the first few pages. Format as an ISO-8601 date in the format yyyy-MM-dd like "2022-03-15"

* **Start and End Validity Dates:** Look for ranges like "Dec 2025 - Apr 2026" or "Jan - Dec 1993". Format as ISO-8601 dates in the format yyyy-MM-dd like "2022-03-15".

**5. Financials:**

* Scan the Executive Summary or "Financial Summary" tables for the **Total Requested** or **Required Budget**.

* Be careful to extract the *Total* amount, not just a sub-sector amount (like Water/Sanitation).

*Just mention the currency symbol ($ instead of US$).

**6. Participating agencies:**

* Extract only the top 5 major agencies or UN bodies (e.g., UNICEF, WFP, UNHCR). Do not list every single NGO partner found in the annex.

**7. Total People in Need and Targeted:**

* Identify the total people in need, and if mentioned, the total people targeted (otherwise 0).

* Be careful to extract the *Total* amount, not just a sub-sector amount (like Water/Sanitation).

## OUTPUT FORMAT

Return strictly valid JSON matching the schema provided. Do not include markdown formatting (```json) or conversational text.
```
This prompt was refined alongside a [structured output schema](https://docs.langchain.com/oss/python/langchain/structured-output) to ensure consistency and compliance to the `Plan` entity's schema. Any changes to either the prompt, schema or model could easily be tested, compared and benchmarked using the platform's "Preview" feature:

![[Pasted image 20260305162932.png|Small changes in the pipeline affect the quality of the resulting extraction]]

After a few iterations, results looked satisfactory on a small subset of the plans, and an automation action was set up to execute the pipeline on all 2,278 plans in batches of 30. After 2h, 30% of the plans were processed and the human review process could begin.

A custom web application was developed ahead of time using the platform to enable human reviewers to easily correct and tweak the extraction. A main page showed all processed plans, and allowed reviewers to filter them by property:

![[Pasted image 20260305163657.png|The Plans Inbox catalogues all processed plans in a familiar interface]]

Clicking on a plan opened up a details page where multiple tools enabled a reviewer to correct LLM-produced properties alongside the scanned PDF, ask a chatbot to find information within the document, and collaborate on the review process by adding comments:

![[Pasted image 20260305164201.png|Reviewing and correcting extracted properties within the platform]]

Once again, the embedded agent was given a carefully-crafted prompt to minimize hallucinations and define the context of its operation:

```markdown
### IDENTITY & PURPOSE
You are **"The Historian,"** the digital guardian of the History Project at OCHA. Your purpose is to help users understand the specific details of Humanitarian Needs and Response Plans (HNRPs) ranging from the 1980s to the present day. You have access to scanned historical documents (some with OCR errors) and modern digital files. Your highest priority is **accuracy**. You do not guess. You do not hallucinate. You only report what is explicitly stated in the text.

### CORE DIRECTIVES
**1. The "Ground Truth" Rule**
Answer ONLY based on the provided document context. If the answer is not in the text, you must state: *"I cannot find that information in this specific document."* Do not use your outside training data to fill in gaps about historical events (e.g., do not write about the Rwanda Genocide unless the document explicitly mentions it). 

**2. OCR Resilience** 
You will encounter "noisy" text from old scans (e.g., "l992" instead of "1992", "US$ 3.5m" reading as "US$ 3,5m"). * **Instruction:** Use context to silently correct obvious OCR typos. * **Instruction:** If a number is illegible or ambiguous in the scan, state: *"The number in the document is unclear due to scan quality."* 

**3. Financial Precision** When asked about budgets or beneficiary numbers: * Always specify the **currency** (usually US$). * Distinguish between **"Requested"** (Asked for) and **"Secured"** (Received) funds if the document makes that distinction. * If a user asks for "The Budget," look for the **"Total Requirements"** in the Executive Summary or Financial Table. 

**4. Terminology Translator** If a user uses a modern term that didn't exist in the document's era, bridge the gap. * *Example:* If the user asks about "Food Insecurity" in a 1985 document, look for terms like "Famine," "Food Deficit," or "Nutritional Failure" and explain: *"The document does not use the term 'Food Insecurity', but refers to 'Food Deficits' in Section 3..."* 

### TONE & STYLE * 
**Objective:** concise and professional. * 
**Archival:** Treat the document as a primary source. Use phrases like *"According to the 1993 Appeal..."* or *"The Executive Summary states..."* *
**Transparent:** If you are summarizing a long section, say *"Summarizing pages 3-5..."* 



### RESPONSE FORMAT 
1. **Direct Answer:** Start with the specific fact requested. 
2. **Evidence:** Quote a short snippet of the text that supports your answer. 
3. **Location:** Cite the page number if available (e.g., *[Page 12]*). 
   
### EXAMPLE INTERACTION 
**User:** "How much money did they want for water?" 
**The Historian:** The 1992 Appeal requests **US$ 2,000,000** for the water sector. 
> "The project would finance procurement of drilling equipment... Total 2,000,000" 
> *[Source: Page 256, Financial Summary]*
```

With insights from this initial extraction in mind, the extraction process was tweaked once more, and the batch process was resumed for the remaining documents.
### 3. Initial analysis
The resulting raw dataset opened up many possibilities for analysis. Some initial findings were [reported](https://unitednations-my.sharepoint.com/:b:/r/personal/cyrus_pellet_un_org/Documents/Microsoft%20Teams%20Chat%20Files/HISTORY%20PROJECT_.pdf?csf=1&web=1&e=rzGTZ3) the subsequent day, though much work remains to be done to extract reliable insights.

![[Pasted image 20260306090635.png|Constructing plots from the raw data uncovered interesting trends]]
## Future steps
With the [raw dataset]([ALL plans extracted metadata TO BE CLEANED.xlsx](https://unitednations.sharepoint.com/:x:/r/sites/OCHAAPMBMonitoringandTools/Shared%20Documents/HPC%20Tools%20and%20Field%20support/HISTORY%20PROJECT/ALL%20plans%20extracted%20metadata%20TO%20BE%20CLEANED.xlsx?d=wb51b6c5450b949ed9548a77d67ab009a&csf=1&web=1&e=YMFhXn&isSPOFile=1&xsdata=MDV8MDJ8fDE5MmQ1Mzg0NmVkNzRmZDY3ZDM2MDhkZTgxMDBhZDJlfDBmOWUzNWRiNTQ0ZjRmNjBiZGNjNWVhNDE2ZTZkYzcwfDB8MHw2MzkwOTAwMzcwMzg2NzczMDJ8VW5rbm93bnxWR1ZoYlhOVFpXTjFjbWwwZVZObGNuWnBZMlY4ZXlKRFFTSTZJbFJsWVcxelgwRlVVRk5sY25acFkyVmZVMUJQVEU5R0lpd2lWaUk2SWpBdU1DNHdNREF3SWl3aVVDSTZJbGRwYmpNeUlpd2lRVTRpT2lKUGRHaGxjaUlzSWxkVUlqb3hNWDA9fDF8TDJOb1lYUnpMekU1T20xbFpYUnBibWRmVFVSRmVsbHFTVFJhUjFsMFRWUkpNVTE1TURCYVJHZDVURlJuTlZwSFRYUlpWRVpxVDBSU2EwOVhVVEZPZWtFd1FIUm9jbVZoWkM1Mk1pOXRaWE56WVdkbGN5OHhOemN6TkRBMk9UQXpNalk0fDFkMTc4MjY2Zjk3YjRjZDc3ZDM2MDhkZTgxMDBhZDJlfDU5OTk1YzI0OWNmNjQ3MjI5OWJiY2ZhOGE4NDY1Mzk1&sdata=SXBmOWttWWhuK3Y2SklvVjVuVmZIVjYzUFMwMW9yTkUySUtmWWthSXdmMD0%3D&ovuser=0f9e35db-544f-4f60-bdcc-5ea416e6dc70%2Ccyrus.pellet%40un.org)) now available, more thorough analyses can be performed on the basis of this data. The pipeline can easily be reproduced using [Langchain](https://www.langchain.com/), [Databricks](https://www.databricks.com/), or other AI-enabled platforms and libraries. The History Project initially aimed to publish older plans (pre-2000s) on ReliefWeb, and the metadata contained in the dataset would greatly help augment the PDF scans themselves.

Establishing a process to insert new documents into the scope of the Project will be crucial in ensuring its legacy, so that future plans will also be preserved for posterity. In an era where web platforms maintained by OCHA come and go, it is more crucial than ever to establish a sovereign space for these documents to live on.