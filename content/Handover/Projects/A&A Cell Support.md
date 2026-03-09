---
status: Ongoing
technical_owner: Jiyeon Park
strategic_owner: Fawad Hussain Syed / Ana Maria Pereira
repository: https://github.com/OCHA-HPCS/aasync
stack: Python (AASync), SharePoint, Teams
---
**Background:** OCHA's mandate includes coordinating response between various partners during natural disasters. This coordination happens within the context of Assessment & Analysis Cells, which are activated upon request by regional offices to receive remote support during assessment all the way through aid delivery. SPPB maintains multiple workflows and tools used during such activations.
## On Activations
Since January 2026, activations come in multiple levels aimed at distinguishing between internally initiated engagement and request-driven activation:

| Status Level | Status Name                                  | Description                                                                                                                                                                           | Triggers                                                                                                                                                         |
| ------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A0           | Information Gathering & Monitoring           | Light activation focusing on early information gathering, baseline data compilation, and monitoring of emerging signals. Activities are exploratory, with limited analytical outputs. | Internally initiated by A&A Cell (independent of formal RO/CO request). Triggered by emerging risk signals, early alerts, or UNDAC pre-deployment.               |
| A1           | Preliminary Situation Awareness & Assessment | Rapid development of initial situation awareness through preliminary analysis of impacts, risks, and trends, including early analytical notes to inform preparedness.                 | Usually initiated following a request or signal from OCHA RO, CO, UNDAC, or coordination leadership, or when early information gaps require structured analysis. |
| A2           | Targeted Analytical Support                  | Focused analytical, assessment, and information management support to emerging field coordination needs, with more regular and tailored outputs.                                      | Initiated at the request of RO/CO, UNDAC, or coordination mechanisms to support early planning, prioritisation, or briefings.                                    |
| A3           | Assessment and Analytical Support            | Full activation delivering comprehensive assessments, in-depth analysis, sustained IM support, and close alignment with operational planning and response prioritisation.             | Formally requested by RO/CO, UNDAC leadership, or HQ, linked to major response activation or sustained coordination demand.                                      |
| A4           | Scale-down & Transition                      | Progressive reduction of support as analytical and coordination demands decrease, focusing on consolidation, handover, lessons learned, and transition to routine monitoring.         | Initiated jointly by A&A Cell and requesting entities once response stabilises or analytical demand reduces.                                                     |
## On Meetings
Meetings are held on a regular basis to allow partners to share updates and request support for their analysis. Partners may also express interests in specific regions or infrastructure, whose analysis could benefit from focused efforts by other partners (in particular UNOSAT and other data-gathering entities). Meeting notes and recordings are kept in the activation's dedicated SharePoint folder (see [examples](https://unitednations.sharepoint.com/sites/2025AAHurricaneseason/Shared%20Documents/Forms/AllItems.aspx)).

![[A&A Cell Meeting Mozambique 26012026.pdf]]
## Products
Reports are often created and shared within the context of A&A cells, whether developed upon request from partners or as part of usual outputs from assessment entities (UNITAR, CSC, PDC, etc...).

![[rain_fcst_20-24_Jan_2026.png]]

Products are seldom emitted by our branch, though several key pieces of information are shared internally with country offices such as [[RAG Development|AI-assisted SDRs]] and [[EPIC Development|EPIC Snapshots]].
## AASync
The [Assessment & Analysis Sync Tool](https://github.com/OCHA-HPCS/aasync) (or AASync) provides a way to download all [DisasterAware](https://disasteraware.pdc.org/) products linked to a hazard. Using OneDrive's desktop client (installed by default on work laptops), products can then be re-synced to any SharePoint folder.
### Setup
AASync runs locally on-demand, and requires a Python interpreter to be installed (a local administrator account is required on work laptops). The following steps should then be followed:
1) Clone the repository or [download](https://github.com/OCHA-HPCS/aasync/archive/refs/heads/master.zip) the code as a zip file
2) Create a `.env` file in the root directory and fill out the `PDC_USER` and `PDC_PASSWORD` environment variables (see `.env.example` for reference)
3) Install required packages with `pip install -r requirements.txt`. A [UV](https://docs.astral.sh/uv/) environment is recommended as a prerequisite
### Usage
In order to sync products to a local folder, this two-step process is followed:
1) Run `python -m aasync list -q [query]`, replacing `[query]` by the name of the hazard to sync (e.g: `python -m aasync list -q Melissa`). A table of matching hazard should appear after a couple of seconds, showing the hazard names and their PDC ID
2) Run `python -m aasync sync [product_id] [output_dir]`, where `[product_id]` is the PDC ID identified in step 1, and `[output_dir]` is the full pathname of the folder in which products will be placed. If `[output_dir]` is a folder located on OneDrive (see [[A&A Cell Support#Syncing with SharePoint|Syncing with SharePoint]]), the products will be uploaded automatically
### Syncing with SharePoint
To sync products to a SharePoint folder, the folder must first be replicated in your OneDrive. In SharePoint online, right-clicking a folder will show an "Add shortcut to OneDrive" option to that effect:

![[Pasted image 20260309102358.png]]

Once this option is clicked, your local OneDrive folder will contain a corresponding folder mapping SharePoint contents (with two-way updates). This local folder can now be used with AASync.
### Making changes
The DisasterAware API has not changed over the past 6 months. Nonetheless, if changes ever needed to be made to AASync, LLM agents can easily understand the existing code and execute those changes, provided instructions are clearly specified.