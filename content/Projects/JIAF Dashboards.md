---
status: Completed
technical_owner: Jiyeon Park
strategic_owner: Ana Maria Pereira
repository: https://github.com/OCHA-HPCS/jiaf-dashboards
stack: Python, Streamlit, Docker
---
**Background:** The [JIAF analysis platform](https://analysis.jiaf.info/) provides answers to several key questions during the JIAF data review process, including sectoral and inter-sectoral needs analyses. Previously, the platform involved two Tableau dashboards constructed using ad-hoc R scripts by Kashif. A [proposal](https://unitednations.sharepoint.com/:w:/s/OCHAAPMB/IQB6hDv4qOGUQZaeYeSg3D6pAW8S38AFUPv19KehxuU7yig?e=jWeJgH) was drafted to come up with a long-term replacement.

![[Pasted image 20260305101201.png|The new JIAF dashboard displaying the PiN(2) page for Venezuela]]
## PowerBI evaluation
For interoperability reasons with the new MS Fabric platform, PowerBI seemed like an obvious choice for this transition. Nonetheless, since the dashboards consist of a large number of maps, using it requires depending on the official ArcGIS plugin for PowerBI. After several attempts to reproduce the Tableau features in PowerBI, several key roadblocks subsisted:
- **COD layers** on choropleth maps **cannot** be specified using native PowerBI parameter fields, meaning every single map has to be re-configured manually within a country dashboard, as well as across different countries.
- Similarly, **geospatial joins on CODs by p-codes** has to be **manually** re-configured for every map (around 30 times per country dashboard).
- **Visual customizations** (pie charts, boundaries, etc...) are **limited** as compared to ArcGIS server or Datawrapper maps.
- Most critically, **ArcGIS maps have been severely broken on published dashboards** for several years (see [this issue](https://community.esri.com/t5/arcgis-for-power-bi-questions/arcgis-maps-for-power-bi-v2024-1-400-completely/td-p/1527276), [this one](https://community.esri.com/t5/arcgis-for-power-bi-questions/issue-with-arcgis-map-layers-not-displaying-in/td-p/1500245#:~:text=After%20the%20latest%20update%2C%20a%20problem%20has,Microsoft%20Edge%20and%20on%20my%20computer%20only.), [this one](https://support.esri.com/en-us/bug/using-a-web-map-service-wms-or-web-map-tile-service-wmt-bug-000152462), and [this one](https://community.esri.com/t5/arcgis-for-power-bi-questions/arcgis-for-power-bi-doesn-t-support-geocoded/td-p/1100241#:~:text=ArcGIS%20for%20Power%20BI%20doesn't%20support%20geocoded,rules%20out%20any%20kind%20of%20area%20map.)), with no indication that these shortcomings will be addressed in the near future. Proposed mitigations did not help.
## The solution: Python / Streamlit dashboards
[Streamlit dashbaords](https://streamlit.io/) provide solutions to all these issues, while still using familiar technologies for OCHA: Python and Excel. The visuals and the data processing are pre-defined in Python, and additional countries can easily be onboarded using their raw JIAF Excel file and a single text-based configuration file. This approach has several advantages:
- No programming / tweaks of any kind are required to support new countries, and Excel files do not have to be adapted / translated / normalized to work with the platform.
- LLMs can be relied upon to write the country configuration file from a given Excel file, making the process completely worry-free.
- The technology is open-source and can be deployed on any platform (Azure, AWS, locally, on the [[HPCS Cluster]], or on OCHA servers).
- Anything can be changed or customized as desired, including switching maps over to ArcGIS or Datawrapper if needed.

The code for these new dashboards is hosted on the [HPCS GitHub account](https://github.com/OCHA-HPCS/jiaf-dashboards).
### Setup
As explained in the repository's `README.md` file, setting up the dashboard involves a few simple steps:
1) Getting [Python](https://www.python.org/) installed on your computer by either:
	1) Installing Python directly via [this link](https://www.python.org/downloads/)
	2) Installing a Python [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment) such as [PyCharm](https://www.jetbrains.com/pycharm/download/)
	%% UN laptops will not allow you to install 3rd party software by default, but an Administrator account can be requested to the IT services %%
2) Downloading the code on your computer (either by using [git](https://git-scm.com/) or using [this zip file](https://github.com/OCHA-HPCS/jiaf-dashboards/archive/refs/heads/master.zip))
3) Installing the required packages specified in `requirements.txt` by executing the following command:
```bash
    pip install -r requirements.txt
```
### Running the dashboards
The dashboards should now be available to run using this command:
```bash
streamlit run app.py
```
A webpage should open, allowing you to select a country. If not, you may need to navigate to `http://localhost:8501` manually using your web browser. If any errors are displayed, make sure packages were correctly installed when following the [[JIAF Dashboards#Setup|setup]] instructions.
### Making changes
The dashboard code is broken down into several files:
- `app.py`: main entrypoint, contains generic functions and a list of pages (see [[JIAF Dashboards#Adding a new page|Adding a new page]])
- `data.py`: contains the functions that load and clean up the JIAF Excel files; pre-processing the data so it is ready to use in all pages (works alongside `etl.py`)
- `etl.py`: contains specific functions to clean up data and load country configurations
- `precompute.py`: script used to convert the Excel file into a [Parquet file](https://parquet.apache.org/) (optional)
- `figures.py`: contains all the code used to process CODs and create maps
- `pages/***.py`: actual dashboard pages (pin, severity, linkages)

Simple modifications to specific pages can be easily coded by prompting an LLM (Copilot, ChatGPT, etc...) like so:

> Make changes to the code provided below to *(explain the change here)*.
> Here is the existing code: *(paste the relevant `pages/***.py` file's contents here)*

For more complex changes, consider using a local agent such as [Claude Code](https://claude.com/product/claude-code) or [Gemini CLI](https://geminicli.com/). These tools can work autonomously and make changes to multiple files at once. The [Streamlit documentation](https://docs.streamlit.io/) and [Pandas documentation](https://pandas.pydata.org/docs/) will be useful to understand how these libraries work and what features are supported. 
### Adding a new country
When a new JIAF country Excel file becomes available, its data can be integrated into the dashboards by following these simple steps:
1) Copy the Excel file over to the root folder of the repository (alongside existing files such as `HTI_Worksheet_3A_3B_PiN&Sev_Compilation_20251101.xlsx`)
2) Create a new file in `config` for the country, and fill out fields as follows:
	1) `name`: the full name of the country (e.g: *South Sudan*)
	2) `file_path`: the filename of the Excel file copied in step 1 (e.g: *South Sudan_Worksheet_3A_3B_PiN&Sev_Template_18NOVEMBER2025 REVIEW 1DEC.xlsx*)
	3) `cod_uri`: the URL (or local file path) of an Admin2 COD dataset for the country, in geojson format. If no high-quality dataset is available, use `https://fdw.fews.net/api/feature/?country_code=SS&unit_type=admin2&format=geojson`, replacing "SS" by the appropriate ISO2 country prefix
	4) `geo`:
		1) `pcode_col`: the name of the column containing the admin2's p-code in the Excel file (e.g: *Admin 2 P-Code*)
		2) `geojson_key`: the name of the COD dataset's p-code field, starting with `properties.` (e.g: *properties.p_code*)
	5) `sheets`:
		1) `pin`: the name of the Excel file's sheet containing PiN data (e.g: *WS - 3.1 Overall PiN*)
		2) `history`: the name of the Excel file's sheet containing historical (past year) PiN data (e.g: *PiN Historical Trend*)
		3) `severity`: the name of the Excel file's sheet containing severity data (e.g: *WS - 3.2 Intersectoral Severity*)
	6) `params`:
		1) `header_row`: row index of the header row containing individual column titles (e.g: *2*)
		2) `start_row`: row index of the first row containing actual data (e.g: *3*)
	7) `sectors`: a list of sectors mentioned in the Excel file **in english** - translations are specified later (e.g: *CCCM, Education, Nutrition, Food Security, etc...*)
	8) `column_mapping`: a list of `English -> Target` translations for the column names **as they appear in the Excel file**. The list of supported English column names can be compiled from the existing examples (COL, HTI, MOZ, SDN, SSD, VEN). If a column is specified differently across sheets, multiple translations can be provided as follows:
	```yaml
	Final Severity:  
	  - "Severidad"  
	  - "Severidad final"
	```

Once the new `config/***.yml` file has been fully defined as described above, the dashboard should be restarted. The new country should automatically appear in the country selector. If anything hasn't been properly defined, specific errors will show from within the pages, and corrections can be made. **Make sure to open all pages of the dashboard to check if anything went wrong**.

> [!TIP] Turbo mode
> Optionally, `python precompute.py` can be run in the command line to optimize data from the Excel file. Running the dashboard again will automatically use the optimized data instead of the Excel file directly. Any changes to the Excel files should be followed by re-running the `precompute` script for changes to take effect.

> [!WARNING] COD Complexity
> Certain COD datasets (including HDX datasets) have a very high geospatial resolution and are hence very large. Detailed datasets do not improve the quality of the dashboard and may significantly increase the dashboard's map loading times. Where possible, use tools such as [this one](https://mapshaper.org/) to download a lower-resolution dataset for dashboard use.
### Adding a new page
Adding a new page to the dashboard involves a straightforward two-step process:
1) Creating a new page script in `pages/***.py`, taking example from the existing 13 pages
2) Modifying the last part of `app.py` to include the new page in the appropriate existing or new section. For example:
```python
pages = {  
    "Overview": [  
        ...
    ],  
    "New Section": [  
        st.Page("pages/new_page.py", title="My new page"),  
    ],  
    "PiN": [  
        ...
    ],
    ...
}
```

Once again, LLMs can be useful tools to generate a new page, providing them with existing pages as an example.
## Severity 5 reviews
In addition to country dashboards, the new JIAF dashboards also aim to replace the legacy [Severity 5 Review PowerBI dashboards](https://unitednations.sharepoint.com/:u:/s/OCHAAPMB/IQAr1wAVzRm0TJRNWins-QKpAXas5Vjp-00eOabOGkpnthk?e=TcUgKw). Detailed information about the Severity 5 review process itself can be found [here](https://unitednations.sharepoint.com/:b:/s/OCHAAPMB/IQA3TXVYig8eTqOBV4kizYjoAcAboOgWkkOP5CWZyHPsXjM?e=pRF3B8). The Streamlit dashboard defined in `severity-5/app.py` automatically fetches data from the Kobo forms referenced in the `surveys` variable, and displays results in several convenient review panels.

Data is synced using the [Kobo API](https://support.kobotoolbox.org/api.html), and a valid API key is needed to make the dashboard work. After creating your own API key in the Kobo settings, you must replace the existing api key in `severity-5/app.py`, line 43:
```python
API_KEY = "0b93ccf6941932f0b1e5cf5bdd9ad69a8fa87cef" # replace this by your own API key
```

Once [[JIAF Dashboards#Setup|setup]] instructions have been followed, the dashboard can be run using the following command:
```bash
streamlit run severity-5/app.py
```

![[Pasted image 20260305114634.png|The new JIAF dashboard showing the survey results of the 2026 Columbia Severity 5 review]]

The dashboard is designed to work out-the-box with no tweaks needed, provided that new Kobo forms match the structure of the existing forms. Within existing sections, questions can be added and removed as needed. As previously mentioned, the only change needed to support a new review is to edit the `surveys` dictionary by adding a new `Country name: Kobo survey ID` entry:

```python
surveys = {  
    "Colombia": "ajiqJ68LUrt4d4v4yRb7gw",  
    "Myanmar": "awizpCCNrU6UiXxcqf4grg",  
    "Sudan": "aX8aLeo8iuxQQazNRKAVKD",  
    "Venezuela": "aMtE6yAAM2XwFptHZYUPNB",  
    "South Sudan": "afzb6qQKd6p7zziT6cTunx",  
    "Haiti": "aKpBUdZZqLmj4wAxBkYgXx",
    # Add new entries here as needed  
}
```

> [!WARNING]
> Forms must be explicitly **published** in Kobo for the dashboard to be able to pick up data
#### Making changes
Modifications can be easily applied to the dashboard by prompting an LLM (Copilot, ChatGPT, etc...) like so:

> Make changes to the Streamlit dashboard provided below to *(explain the change here)*.
> Here is the existing code: *(paste the entire `app.py` file's contents here)*
## Deploying the dashboards
The dashboards are bundled with [Dockerfiles](https://www.docker.com/) and [Kubernetes](https://kubernetes.io/) manifests for industry-standard deployments on any platform or cloud that supports containerization. A [GitHub workflow](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows) is also set up on the repository to automatically build [Docker images](https://github.com/orgs/OCHA-HPCS/packages?repo_name=jiaf-dashboards) of the country and severity-5 dashboards within 2 minutes of changes being pushed.

These deployment mechanisms should remain stable for at least the next five years and will not require any changes to keep working as the project keeps being developed. Azure is [able](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart) to deploy containers automatically, and can hence be used as soon as OCHA gains access to the console. Low-cost cloud providers such as [Hetzner](https://www.hetzner.com/) also provide one-click deployments for Docker containers. Lastly, the [[HPCS Cluster]] has successfully hosted the dashboards in the past, and contains all the necessary configurations to enable them to work out the box.