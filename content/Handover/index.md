---
title: Handover Notes
---
**Intern – Strategic Prioritization & Practices Branch (SPPB)**  
**Humanitarian Programme Cycle (HPC) Section**  
**Assignment period: 1 October 2025 – 31 March 2026**
## Executive Summary
Between October 2025 and March 2026, I supported the Strategic Prioritization & Practices Branch (SPPB) within the Humanitarian Programme Cycle (HPC) Section, primarily contributing to the development and deployment of Information Management tools, dashboards, and analytical support platforms.

Key achievements include:
- Deployment of JIAF severity 5 dashboards across six countries
- Development and partial institutionalization of the RAG architecture
- Operational support to A&A Cell activations through analytical and coordination tools
- Deployment of EPIC during the 2025 hurricane season
- Establishment and maintenance of the HPCS cluster hosting multiple platforms
- Rollout of AIR-AIS scripts supporting the ActivityInfo Country Module
- Continuation and significant advances for David Goetghebuer's History Project

Several platforms are operational but require:
- Clear technical ownership
- Dedicated maintenance capacity
- Strategic alignment with OCHA’s broader transition to Microsoft platforms

Immediate priorities (April–June 2026):
- Clarify ownership of RAG and EPIC
- Secure funding pathway for A&A tools
- Ensure HPCS cluster sustainability within Azure governance
- Document and formalize JIAF dashboard maintenance workflows
## 1. Role Overview
Supported Information Management platforms development, dashboarding, and inter-partner collaboration across HPC-related initiatives, including analytical support to A&A Cell and development of prioritization and monitoring tools.
## 2. Key Contacts
**Supervisors:**
1) Ana-Maria Pereira pereira9@un.org
2) Fawad Hussain Syed fawadhussain@un.org
3) Nick Imboden imboden@un.org

**Key working relationships**:
1) HPC Team members: Ana-Maria Pereira, Denise Pinto, Fawad Hussain Syed, Herbert Tatham, Jiyeon Park, Leyri Segura Gomez, Lilian Barajas, Magalie Salazar, Natthinee Rodraska, Rachel Maher, Tinago Chikoto, Uta Filz and Ysabel Fougery in relation to [[HNRP Typesetting]], [[JIAF Dashboards]], [[RAG Development]], [[A&A Cell Support]], [[EPIC Development]], [[HAVEN Development]] and [[HPCS Cluster]]
2) Monitoring Team members: Bassil Aleter, Kashif Nadeem and Nick Imboden in relation to [[AIR-AIS Development]]
3) A&A Cell members from ACAPS, Atlas Logistique, CHD, DFS, EC JRC, ECHO, ERCC, EUCP, FAO, FIS, HKV, IFRC, IMMAP, IMPACT, IOM, Kobo, MapAction, OHCA, PDC, ROLAC, ROMENA, ROSEA, ROWCA, RVO, UNDAC, UNEP, UNHCR, UNICEF, UNITAR, UNOSAT, WFP-ADAM and WHO in relation to [[A&A Cell Support]] (see [Contact list 2025.xlsx](https://unitednations.sharepoint.com/:x:/s/2025AAHurricaneseason/IQBa4ONzNIefS4iD4X22r_KHARCOR9V3hHv8PzEIsuVgrnI?e=XAFmoS)).
## 3. Project Status Updates
### [[HNRP Typesetting]]

> [!INFO] Initial survey completed (November 2025)
> Trialed during Oct-Nov 2025. The InDesign plugin approach was ruled out, and nothing was definitely put in place for this year. The same roadblocks will certainly be raised in the Fall of 2026.
### [[JIAF Dashboards]]

> [!SUCCESS] Successfully deployed (December 2025)
> Dashboards for Columbia, Haiti, Mozambique, Sudan, South Sudan and Venezuela were successfully deployed in December 2025. Several severity 5 reviews for those same countries were successfully performed using the accompanying severity 5 dashboards. Kashif's departure will make these new dashboards essential tools in the future.
### [[Work/OCHA/Handover/Projects/History Project|History Project]]

> [!DONE] Successfully completed (January 2026)
> 2,278 inter-agency plans from 1992 onwards were collected, scanned and processed using LLMs to extract key data points (PiN, clusters, crisis type, etc...). Initial analysis was performed, and the raw dataset is available for further use.
### [[RAG Development]]

> [!INFO] Development ongoing (January 2025)
> A large number of Secondary Data Reviews (SDR) were issued through local use of the tool. Several key projects (Frontend, Dockerization, API) were completed to enable deployments across OCHA. The next generation of Graph-RAG architecture was developed and is awaiting implementation.
> 
### [[A&A Cell Support]]

> [!INFO] Development ongoing (February 2026)
> Several tools such as AASync have been used on a regular basis during all A&A cell activations. More comprehensive collaboration tools are being drafted out, and will require dedicated funding to bring to fruition. Current platforms provide insufficient features/engagement, and leave market space for a replacement.
### [[EPIC Development]]

> [!CAUTION] Development paused (December 2025)
> EPIC was successfully deployed and operated throughout the 2025 hurricane season. High technical complexity and development requirements did not allow further work and partner onboarding for the time being. Dedicated funding will be required to maintain the platform in the future.
### [[HAVEN Development]]

> [!DONE] Successfully deployed (March 2025)
> A working prototype was deployed and trialed. Full integration with Datawrapper (charts & maps alike) was achieved. Minimal involvement will be required to maintain the project.
### [[AIR-AIS Development]]

> [!DONE] Successfully deployed (March 2025)
> ActivityInfo Scripts (AIS) are a critical component to the recent successful rollout of the new ActivityInfo Country Module. Regular maintenance and upgrade of these scripts has been planned for, and will enable continued operations independently from BDD's roadmap.
### [[HPCS Cluster]]

> [!DONE] Successfully deployed (November 2025)
> Several projects such as RAG, EPIC, the JIAF dashboards and HAVEN were hosted on the cluster, with a maximum monthly spending of 17.65 USD. Maintaining the cluster and its specification will be key to future platforms' success in OCHA's Azure era.
## 4. Key Processes & Workflows
### JIAF Processing Workflow
1. **Data Collection:** Obtain the latest JIAF Excel workbook from the country office.
2. **Configuration:** Create/Update a YAML configuration in `config/` mapping the Excel sheets, column headers (PiN, Severity, etc.), and Admin2 P-codes (see [[JIAF Dashboards]]).
3. **Validation:** Run `streamlit run app.py` locally to verify that all maps and figures render correctly.
4. **Severity 5**: Create the corresponding Severity 5 Kobo form for review (if needed).
5. **Deployment:** Push changes to GitHub to trigger the automated Docker build and deploy to the [[HPCS Cluster]].
### A&A Cell Activation Workflow
1. **Monitoring:** Regular tracking of hazard events (e.g., hurricane paths via GDACS/PDC).
2. **Triggering:** Formal activation of the A&A Cell upon reaching severity thresholds.
3. **Synchronization:** Use `AASync` scripts to ensure field data and partner reports are centralized in SharePoint/Teams.
4. **Coordination:** Convene multi-partner meetings; document minutes in the relevant SharePoint folder.
5. **Product Aggregation:** Consolidate analytical products from various sources for AI-assisted SDR use (see [[RAG Development|OCHA RAG]]).
### HPCS Cluster Deployments
1. **Dockerization:** Ensure the application to be deployed has a multi-stage `Dockerfile` optimized for size and security.
2. **Manifests:** Maintain Kubernetes YAMLs for `Deployment`, `Service`, and `Ingress` (with SSL/TLS via Let's Encrypt).
3. **CI/CD:** Utilize GitHub Actions to build and push images to the `OCHA-HPCS` container registry.
4. **Cluster Management:** Use `terraform` to apply changes to the Rackspace Spot environment; monitor spending via the Spot console to maintain the <$20/month target.
## 5. Meeting Documentation

### HPCS & HPC-Focused Meetings
Occurring every Tuesday and Thursday respectively, notes being taken by interns taking turns at [2025 - NARAS Meeting Minutes.docx](https://unitednations.sharepoint.com/:w:/s/OCHAAPMB/IQBRhnZREDV0TYlx96RUsbkaAdrq3kQ8pVs9rOro47VEToY?e=TLyyTW). Copilot can provide transcripts and summaries if recordings are allowed. Attendance and action points are included; these notes being useful for catching up when unable to attend.
### A&A Cell Meetings
Occur every few days during A&A Cell activations, involving both UN agencies and external partners (NGOs, etc...). Recordings are to be managed, and notes included in specific activation folders (see [[A&A Cell Support]] and [[index#4. Key Processes & Workflows|Key Processes & Workflows]]). Examples can be found in [A&A Cell meetings](https://unitednations.sharepoint.com/sites/2025AAHurricaneseason/Shared%20Documents/Forms/AllItems.aspx?httpstatus=undefined&contentlength=undefined).
## 6. Tools & Access
### SharePoint sites
%% Access pre-granted for most sites. Specific sites (A&A, AI) shared upon request %%
- [APMB](https://unitednations.sharepoint.com/sites/OCHAAPMB): Main SPPB site including MATS, HPCS and APS colleagues. Contains recent GHO, Humanitarian Reset and HPC CoP documentation
- [OCHA Hub]( https://unitednations.sharepoint.com/sites/OCHAHub): Contains links to various OCHA platforms, as well as inclusion, funding and HR-related pages, as well as brand templates and guidelines
- [A&A Cell Coordination](https://unitednations.sharepoint.com/sites/2025AAHurricaneseason): Not set up as a site, but as a drive and teams channel for [[A&A Cell Support]]. Should be enhanced and made more presentable
- [OCHA AI](https://unitednations.sharepoint.com/sites/OCHA-AI): Primarily provides high-level corporate guidance on AI use; not intended as technical documentation or implementation support
- [JIAG](https://unitednations.sharepoint.com/sites/JIAG-JointIntersectorAnalysisGroup): Contains a wealth of older documents about the JIAF framework, including trainings, concept notes, etc... Doesn't currently seem to be kept updated
- [OCHA HPC](https://unitednations.sharepoint.com/sites/OCHAHPC): HPCs site containing recent HPC-related CoP documents, clinics and workshops information, and links to the latest HPC tracker
- [HPC](https://unitednations.sharepoint.com/sites/HPC-HumanitarianProgrammeCycle): Legacy site containing documentation from 2024 and earlier; materials may not reflect current HPC guidance
- [OCHA IM](https://unitednations.sharepoint.com/sites/OCHAIMB): Information Management-specific site. Includes analytics for ReliefWeb
### Teams channels
%% Access usually pre-granted through meeting invitations or SharePoint site access %%
- [Needs Analysis Unit Chat | Group Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:9db0f6a72ec14138b16d742694ab96af@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [HPC Section Weekly Meeting | Meeting Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:meeting_MjcyNDk3MjYtMzc4Zi00YjY3LTlhNmQtNzFmYWY1NWNiMjk0@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [Natural disasters - SPPB internal | Group Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:c44b83ff46304b1caf00f3380c7809d9@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [SPPB OPS Meeting | Meeting Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:meeting_NjU3NTcxMDctYWMzYi00NmQzLWI5ZDUtMTVhM2Q4ZjU5MGEx@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [HPC-Focused Meeting | Meeting Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:meeting_YjYwNjU2MzAtZGFjMy00YWZiLWE0YTktZWVhNzczZTZkYmE3@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [HPC Monthly catch-up call | Meeting Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:meeting_MDY1NmViNjMtMTQxYy00MWQxLTg2NTgtODhlOTM1NzZmYTBl@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
##### OCHA-Wide interns channels
%% Access granted by any current intern %%
- [The Cool Interns | Group Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:22074781137d49e9b33781a21ad4fb4e@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [The Lunch Bunch | Group Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:4a978ddd73ac425481c89b50b29e13d1@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [OCHA Interns | Group Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:10a1276484464417b4231aa1509e6355@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
- [OCHA Interns | Group Chat | Microsoft Teams](https://teams.microsoft.com/l/chat/19:e26f875f1ff84abc9b69d7790e636ea4@thread.v2/conversations?context=%7B%22contextType%22%3A%22chat%22%7D)
### Software Engineering tools
%% OCHA access granted by Andrej Verity. HPCS through Fawad %%
- [OCHA HPCS GitHub Organization](https://github.com/OCHA-HPCS)
- [OCHA GitHub Organization](https://github.com/UN-OCHA)
- [Cyrus Pellet's GitHub](https://github.com/cpellet)
### Information Management Software
%% Access through UN email address, or by registering manually %%
- [PowerBI](https://app.powerbi.com/) installed through IT or via local root account
- [ArcGIS](https://gis.unocha.org/portal/home/): Maps and geospatial datasets (CODs, etc...)
- [HDX](https://data.humdata.org/): Humanitarian datasets maintained by OCHA
- [Kobo](https://kobo.unocha.org/): Survey software used for field collection, sev 5 reviews (see [[JIAF Dashboards]]), etc...
- [PyCharm](https://www.jetbrains.com/pycharm/): My recommended Python IDE
### A&A Cell platforms
%% Access by registering manually, where needed %%
- [DisasterAware](https://disasteraware.pdc.org/): Most useful real-time aggregation platform for products across partners
- [GDACS](https://www.gdacs.org/): EU/UN preparedness dashboard
- [SMCS](https://smcs.unosat.org/home): GDACS Satellite Mapping Coordination System
- [USGS](https://www.usgs.gov/): US Geological Survey platform
- [Copernicus](https://mapping.emergency.copernicus.eu/): EU mapping platform
- [Logie](https://logie.logcluster.org/?op=brb): Logistics cluster dashboards
- [Virtual OSOCC](https://vosocc.unocha.org/) (not actively used)
- [[EPIC Development|EPIC]]
### HPCS-specific
%% Access through UN email address %%
- [HPC Knowledge Platform](https://knowledge.base.unocha.org/wiki/spaces/hpc/overview?homepageId=3971612986)
## 7. Lessons Learned & Strategic Observations
1) **Tool Proliferation vs Institutionalization**  
    Many tools are successfully piloted but lack long-term ownership or maintenance pathways.

2) **Funding Gap for Analytical Innovation**  
    Several platforms (EPIC, A&A tools, RAG evolution) require modest but dedicated funding to transition from prototype to institutional asset.

3) **Cluster-Based Hosting Is Cost-Efficient**  
    The HPCS cluster demonstrated that lightweight hosting can be both affordable and operationally effective, instead of relying on costly Azure services.

4) **Engagement During Activations Is Strong but Unsustained**  
    A&A Cell participation is high in early activation phases but declines without structured collaboration tools.

5) **Dashboards Are Increasingly Mission-Critical**  
    With Monitoring Team capacity changes, JIAF dashboards will likely become core institutional tools rather than supplementary products.

6) **Documentation Is a Structural Weakness**  
    Processes rely heavily on tacit knowledge and intern continuity.
## 8. Recommendations for Continuity
**Short-Term (Next 3 Months)**
- Assign formal technical owner for RAG and EPIC
- Document HPCS cluster governance and cost center
- Formalize JIAF dashboard update SOP
- Consolidate A&A Cell documentation structure

**Medium-Term (6–12 Months)**
- Explore funding pathway for A&A collaborative platform
- Integrate RAG architecture into broader OCHA AI strategy
- Align HPCS hosting with OCHA Azure enterprise standards

**Structural**
- Reduce reliance on intern continuity for core platforms
- Establish product lifecycle management approach for digital tools
## 9. Contact information 
Cyrus Pellet cyrus.pellet@gmail.com.
%% Available for follow-up questions %%
## Annex: Project ownership snapshot

| Platform         | Status         | Technical Owner    | Strategic Owner    |
| ---------------- | -------------- | ------------------ | ------------------ |
| HNRP Typesetting | In development | Jiyeon Park        | Ana Maria Pereira  |
| JIAF Dashboards  | Completed      | Jiyeon Park        | Ana Maria Pereira  |
| History Project  | Completed      | Sarah Choong       | Sarah Choong       |
| OCHA RAG         | In development | Fawad Hussain Syed | Fawad Hussain Syed |
| EPIC             | Interrupted    | Fawad Hussain Syed | Fawad Hussain Syed |
| HAVEN            | Completed      | Fawad Hussain Syed | Fawad Hussain Syed |
| AIR              | In development | Nick Imboden       | Nick Imboden       |
| AIS              | Completed      | Nick Imboden       | Nick Imboden       |
| HPCS Cluster     | Interrupted    | Fawad Hussain Syed | Fawad Hussain Syed |
