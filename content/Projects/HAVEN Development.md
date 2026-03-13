---
status: Completed
technical_owner: Fawad Hussain Syed
strategic_owner: Fawad Hussain Syed
repository: N/A
stack: Python, Datawrapper API
---
**Background:** Most charts in our section are created using MS Office, ArcGIS or Datawrapper. The latter produces a range of pre-configured high-quality visualizations (including maps), but doesn't come without downsides. Its lack of integration with Excel requires IMs to copy-paste data into Datawrapper's online editor, often needing to correct individual cells and tweak columns before obtaining the desired results. [Generative UI](https://research.google/blog/generative-ui-a-rich-custom-visual-interactive-user-experience-for-any-prompt/) can provide a novel solution to these shortcomings.

![[3MoiU-projected-vs-gho-prioritised-population-pp-by-country-.png|Darawrapper chart illustrating GHO projected and prioritized figures by country|600]]
## Humanitarian Analytics & Visualization Engine for Needs
HAVEN consists of a spreadsheet and a chatbot working in sync to create visualizations using Datawrapper. Existing data can be uploaded as an Excel or CSV file. The chatbot has full visibility over the different sheets, and can read/manipulate/create values or formulas in any cell. This allows it to dynamically prepare the data in the correct format for Datawrapper, no matter the state of the workbook it was contained in.

![[Pasted image 20260309151549.png|The HAVEN interface, including the chatbot and the spreadsheet]]

Beyond generating visualizations, HAVEN can also perform other tasks on spreadsheets, including:
- Clean up data according to instructions
- Pivot/unpivot columns, compute aggregations, and insert formulas
- Operate on multiple sheets at once
## Technical details
At its core, HAVEN is simply a wrapper around commercial LLMs such as GPT and Gemini, including a set of [custom tools](https://www.ibm.com/think/topics/tool-calling) to interact with the spreadsheet and [dynamically fill out UI elements](https://tambo.co/). The spreadsheet interaction was inspired from [this project](https://github.com/michaelmagan/cheatsheet), and the Datawrapper integration was coded as a custom MCP server.
## Potential future directions
- **Azure Integration:** Explore porting custom MCP tools to Microsoft's Power Platform / Copilot ecosystem.
- **HDX/ReliefWeb Integration**: Pulling HNRP/GHO data directly into the spreadsheet
- **ArcGIS Integration**: Fetching OCHA-standard CODs for more accurate spatial joins during map-making