---
status: In development
technical_owner: Jiyeon Park
strategic_owner: Ana Maria Pereira
repository: N/A
stack: InDesign, Python (trialed)
---
**Background:** Producing HNRPs requires country offices to manually edit an OCHA-provided [template](https://unitednations-my.sharepoint.com/personal/pereira9_un_org/Documents/Microsoft%20Teams%20Chat%20Files/HNRP_2026_template_annotated_FINAL.pdf "https://unitednations-my.sharepoint.com/personal/pereira9_un_org/Documents/Microsoft%20Teams%20Chat%20Files/HNRP_2026_template_annotated_FINAL.pdf") using [Adobe InDesign](https://www.adobe.com/products/indesign.html). HNRP infographics are produced separately using [Adobe Illustrator](https://www.adobe.com/products/illustrator.html), and need to be manually linked into the InDesign project at country level. The whole process is lengthy and error-prone, involving a lot of copy-pasting and manual fixing to maintain the layout of the pages.

3 approaches were explored to remedy these issues:
## 1. Python scripts
%% (see Ana Maria's work on this) %%

> [!NOTE]
> InDesign project files come in two distinct variants: an old [XML](https://www.w3schools.com/xml/xml_whatis.asp)-based format called `.idml` and a new proprietary format called `.indd`. Here, we export projects as `.idml` as this allows us to rename them as `.zip`, and edit all of their XML contents after unzipping them, without involving InDesign itself at all.

The goal of this approach is to replace chunks of text from the template with actual country text without involving InDesign, thanks to some custom Python scripts. The process roughly works as follows:
1) Country offices define the contents of their HNRPs by filling out specific Word and Excel documents
2) A script is given access to these documents, as well as the `.idml` InDesign template
3) The script looks for specific chunks in the `.idml` file, and replaces those with contents from the Word file
4) The script also optionally patches Illustrator files with the Excel data to produce visualizations
5) The script outputs a new `.idml` file with the customized HNRP

The advantages of this approach are that all text content can be filled out instantly (given that the Word document was filled out properly). However, this does not include images, charts and other infographics, and the resulting file still requires a fair amount of manual labor to fix the layout in InDesign.

**All scripts and test templates can be found [here](https://unitednations-my.sharepoint.com/:f:/g/personal/cyrus_pellet_un_org/IgCfFEU6eQgtQYAQQsyvoroLAZJplkQ8akDV_lbwFN09nDQ?e=yvvPS1).**
## 2. InDesign plugin
This approach involves writing a custom InDesign extension (in JavaScript) to automatically fill out the template from InDesign itself, without having to run scripts.

Even though a [working prototype](https://unitednations-my.sharepoint.com/:f:/g/personal/cyrus_pellet_un_org/IgB6DO3527cQR40VOYJPuzkPASsAFwUh5RXtn7aJlBIGNmM?e=Ohscpo) was developed, this approach is **not recommended** for a few reasons:
1) OCHA does not have the capacity to maintain yet another tool, especially not in JavaScript
2) Adobe products are notably unstable and tend to change regularly, which means the tool will often need to be updated
3) This still doesn't cover all use cases: infographics and charts still need to be generated elsewhere, and layout issues are difficult to detect and fix automatically
## 3. Typst
[Typst](https://typst.app/) is a new typesetting language and software that enables direct and predictable document generation using code. Tables, charts, visualizations and more are supported out the box, and documents are infinitely customizable.

![[Pasted image 20260304094321.png|280]]![[Pasted image 20260304094341.png|390]]
%% The elements above were entirely reproduced using Typst (see [here](https://typst.app/project/r8sHwRal2qiZQZioa480Ch))%%

For this approach, we ditch InDesign entirely and write the HNRP template in [Typst syntax](https://typst.app/docs/tutorial) instead, using [variables](https://typst.app/docs/tutorial/making-a-template/) for country-specific content. This enables country offices to fill out the template using the simplest method so far: providing a list of values corresponding to each variable, **including data for visualizations**, which are generated on the fly.

This brings several benefits:
1) The template is defined once, then no specific skills are required by country offices to produce their HNRPs at will
2) Specific customizations can easily be applied if needed, either by country offices upon request, or across years if the design needs to be tweaked
3) The whole process (including [visualization generation](https://lilaq.org/)) takes under 1s, enabling real-time feedback of how the final product looks like
4) Brand guidelines are enforced by default, so that country offices can focus on their content instead
5) Localization (including RTL support for the Arabic locale) is supported by default, and content can be translated automatically using LLMs, as it is simply defined as a text file
6) The whole technology stack is open-source, and costs nothing to operate, leading to 10,000s-100,000+ CHF of savings in costly Adobe licenses

Reproducing the HNRP template in Typst takes 1-2 months of work full-time, but ultimately saves up many precious months of labor for country office colleagues, who can better spend their time on their actual data and content.

The proof of concept code can be found [here](https://typst.app/project/r8sHwRal2qiZQZioa480Ch).

> [!SUMMARY] After careful consideration of the problem and much time spent thinking about solutions, I highly recommend future involvement in this to seriously consider this Typesetting approach for the reasons listed above

