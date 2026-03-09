---
status: Ongoing / Completed (AIS)
technical_owner: Nick Imboden
strategic_owner: Nick Imboden
repository: https://github.com/UN-OCHA/activity-info-scripts
stack: Python, Typer, Docker, ActivityInfo API
---
**Background:** The Branch's Monitoring team has historically been maintaining two separate platforms: [RPM](https://www.unocha.org/publications/report/world/hpc-tools-rpm-response-planning-and-monitoring-tool) and the [Project Module](https://projects.hpc.tools/). These internally-developed legacy platforms enabled country offices to register partners and activities for reporting purposes. Due to shifting funding conditions, a commercial alternative was selected as a unified replacement.
## On ActivityInfo
[ActivityInfo](https://www.activityinfo.org/) is now used by OCHA and partners to record **cluster activities** in a well-defined ontology for monitoring purposes. Country offices register projects during a planning phase, defining **caseloads**, **activities**, their related **costs** and **indicators**. These projects may be part of broader **coordination plans** associated with HNRPs or the GHO.

The Activities ontology takes its roots in a centralized database called the **Global Reference Module**, which extensively defines a shared library of **entities**, **dimensions**, **user levels**, **cost channels**, **countries**, **age groups**, **plans**, **indicators**, and most importantly **standardized activities** that country offices can refer to and extend at their own will.

Each country office operates within its own replicated database, in which many parameters of the ontology can be tweaked. Notably, **metrics**, **segmentation & disaggregation dimensions**, **coordination levels**, **currencies**, **locations**, **modalities**, **population groups**, and more are country-specific and are configurable by editing table entries in special **configuration forms**.

ActivityInfo schemas support cross-references with external forms, possibly in foreign databases (as is the case with the GRM), as well as the concept of calculated fields, where a basic scripting language allows the user to compute the value of a cell based on form-scoped references and basic math and logic operations. Interestingly, lookups and aggregations are not supported.

![[ActivityInfo_Screenshot.png|Global Logframe reference forms are part of the Global Reference Module]]
### Core caveats
OCHA's use case involves meta-reconfiguration whereby contents of a form's table should specify the schema of another form, due to country office users not being granted write access to form schemas to avoid unwanted changes. This is unsupported by nature in vanilla ActivityInfo, as schemas are defined manually in the UI or through an API, but cannot be constructed dynamically.

Secondly, some of OCHA's use cases involve complex "XLOOKUP"-like formulas. As mentioned previously, lookups and aggregations have not been programmed in ActivityInfo's scripting language, and repeated attempts to expedite the resolution of this blocking point have been unsuccessful.

Thirdly, making changes to the ontology after databases have been replicated for each country office is an extremely tedious process: replicated databases do not keep any "links" to their originating parent, and translations are not carried over through copy operations. Every single change to the master database has to be replicated by hand on every country office database.
## AIR: The Activity Info Runner
The [initial solution](https://github.com/UN-OCHA/activity-info-runner/commit/3b9da89a88ccbf3337529f4ac85dc7ab34b7e1af#diff-b5a26afe06f073f2a4cfc6596eae83503b38003154d138890f4d3b444d922645) to these issues were bespoke scripts targeted towards ensuring business-specific links between forms. Schema formulas (**Internal Calculations**) are defined from a remote table, and can refer to multiple forms from one configuration. More complex expressions are supported via **External Calculations**, in which case a custom parser and AST define a superset of ActivityInfo's internal scripting language to support lookup and aggregations.

![[Pasted image 20260306093205.png|Components involved in AIR's reconciliation loop]]

Issues arose when dealing with configuration field deletions, in which case the side-effects of the scripts had to be reverted. The complexity of the [runner](https://github.com/UN-OCHA/activity-info-runner) quickly increased to reach the level of a fully general reconciliation loop akin to Kubernetes' reconciler's inner workings. Another constraint became supporting arbitrary user-defined scripts, which necessitates standardized inputs and outputs for each step of the process. The following script flow was defined:
1) The user script defines, in generic terms (regex expressions), what the "scope" of the script is on the database tree. We call that scope the **Script Boundary**
2) The runner computes a **Materialized Boundary** from the Script Boundary: the scope becomes expressed exhaustively in currently existing databases, forms, fields and records.
3) The user script computes its Desired Schema from this Materialized Boundary
4) The runner computes a changeset (tuple of database, form, fields and record actions)
5) The runner group changes by API calls and executes the changes
 
![[AIR_Screenshot_2.png|A field changeset is produced in AIR as part of the execution of the Operations Calculations Formulas script]]
## AIS: Activity Info Scripts
Due to a greatly increasing complexity, AIR was put on hold to bring back focus on targeted scripts, while still keeping the underlying AIR API client. The [ActivityInfo Scripts](https://github.com/UN-OCHA/activity-info-scripts) repository provides a minimal scaffolding around logic handling SPPB-specific use cases, and still enables anyone to extend the CLI with custom commands. Currently supported scripts are as follows:

| Name                          | Purpose                                                                                           | File              |
| ----------------------------- | ------------------------------------------------------------------------------------------------- | ----------------- |
| Translations transfer script  | Copy translations for a specific locale from a DB to another, where schemas are assumed identical | `translations.py` |
| Bulk user add script          | Register users from an input Excel file to a target DB                                            | `users.py`        |
| Create data forms script      | Create data forms from form 0.1.2, including basic record fields                                  | `forms.py`        |
| Create reference forms script | Create operation reference forms from form 0.1.3, including basic record fields                   | `forms.py`        |
| Metric config script          | Adjust metric fields from form 0.3.3 in data forms                                                | `config.py`       |
| Disaggregation config script  | Adjust disag fields from form 0.3.2 in disag data forms                                           | `config.py`       |
| Segmentation config script    | Adjust segmentation fields in CDE, LFE, IND/CST/CSL and data forms from form 0.3.1                | `config.py`       |
Scripts are packaged as a self-documenting [Typer](https://typer.tiangolo.com/) command line interface (CLI) tool, and include comprehensive integration tests using ActivityInfo's [Docker image](https://hub.docker.com/r/activityinfo/activityinfo). 

Usage instructions are included in the repository's `README.md` file. Notably, dependencies are defined in the `pyproject.toml` file, and should be pre-installed in a [UV environment](https://docs.astral.sh/uv/).
### Script definitions
A script in AIS is defined by a single Python function decorated by Typer's `@app.command` decorator. It is recommended to specify a `help` string to enable self-documentation, and set `no_args_is_help` to `True` to allow users to see the script's required syntax when running it with no parameters.

A parameter can either be an `Argument` or an `Option`. The former take in a value inline when run (e.g: `python -m main.py config metric xxx` where `xxx` is the target database id), while the latter can be optionally specified as a flag (e.g: `python -m main.py config metric xxx --remove-fields`, where `--remove-fields` sets the `remove_fields` variable to true).

Arguments are defined as follows:
```python
target_database_id: #name of the variable
	Annotated[
		str, #type of the variable
		typer.Argument(help="The ID of the target database") #help message
	]
```

Options are defined as follows:
```python
remove_fields: #name of the variable
	Annotated[
		bool, #type of the variable
		typer.Option(help="Remove existing fields missing from the config") #help message
	] = False #default value when not specified
```

In order to interact with ActivityInfo, a utility was defined to automatically construct a client with typed API methods: `get_client()`, imported with `from utils import get_client`. `client.api` contains all supported ActivityInfo methods, which can be called synchronously as Python functions (e.g: `client.api.get_database_tree(target_database_id)`). These functions are defined in `api/endpoints.py` (see [[AIR-AIS Development#Updating the ActivityInfo client|Updating the ActivityInfo client]]).

The tool was designed to provide real-time feedback to the user about its execution progress through the [Rich](https://github.com/Textualize/rich) package. Its syntax can be inferred from its usage in all current scripts, and its inclusion in new scripts is completely optional.
### Creating new scripts
Registering a new script involves the following steps:
1) If a new group is required (if the script doesn't fall within one of the existing categories: `translations`, `users`, `forms`, `config` and `db`), create a new file `foo.py` (changing `foo` appropriately) and insert the following boilerplate code:
	```python
	import typer
	
	app = typer.Typer(no_args_is_help=True)
	```
	In `main.py`, register the new category by adding `import foo` at the top of the file and the following snippet below existing `app.add_typer` lines:
	```python
	app.add_typer(foo.app, name="foo", help="A useful description")
	```
1) In the new or existing file, append the following script boilerplate:
```python
@app.command(help="A useful description", no_args_is_help=True)  
def my_command(my_arg: Annotated[str, typer.Argument(help="Required argument")], 
           my_optional_arg: Annotated[  
               Optional[str], typer.Argument(help="Argument (optional)")] = None,  
           my_option: Annotated[bool, typer.Option(help="Option")] = False):  
    client = get_client()
    pass #replace this with actual script logic
```

You should now be able to display the script's documentation with `python -m main foo my_command`.
### Making changes with LLMs
LLMs can easily create and modify scripts, provided the scope of the desired changes is well-defined. This setup is recommended to get the best results:
1) Using agents such as [Claude Code](https://claude.com/product/claude-code) or [Gemini CLI](https://geminicli.com/) so that LLMs can read the entire repository before proceeding
2) Providing agents with a complete specification (including input parameters, relevant ActivityInfo endpoints and the expected behavior)
3) Prompting the agent to follow [Red/Green Test-Driven Development](https://www.codecademy.com/article/tdd-red-green-refactor) (Red/Green TDD) to make the changes: a) Writing a test suite for the expected behavior b) Writing the actual script c) Making changes to the script until the tests pass (without ever tweaking the tests)
### Updating the ActivityInfo client
ActivityInfo's API **isn't** documented using a standard `OpenAPI` specification, forcing us to maintain schema parity manually in our codebase.

The data transfer objects involved in API operations are defined in `api/models.py`, and all inherit [Pydantic](https://docs.pydantic.dev/latest/)'s `BaseModel` class to ensure they are deserialized properly from json.

The individual API operations (`GET`/`POST`/`DELETE`) are defined in `api/endpoints.py`. For an operation that returns a struct (i.e: `GET` operations), the function is structured as follows:
```python
def get_user_databases(self) #name of the function (will be used as client.api.get_user_databases)
	 -> List[Database]:  #type of the return value (using models.py)
    raw = self._http.request("GET", "databases") #type and path (/databases)
    try:  
        return [  
            Database.model_validate(item)  
            if isinstance(item, dict) else item  
            for item in raw  
        ]  #validating the json as a list of structs
    except ValidationError as e:  
        raise APIError("Item does not match Database schema") from e #error message
```

For `POST` operations requiring a body as input, the functions are structured as follows:
```python
def update_database_user_role(self, #name of the function (will be used as client.api.update_database_user_role)
	database_id: str, 
	user_id: str, #input parameters
	dto: UpdateDatabaseUserRoleDTO): #model for the json body
    self._http.request(  
        "POST", #type
        f"databases/{database_id}/users/{user_id}/role", #dynamically constructed path
        json=dto.model_dump(  
            mode="json",  
            exclude_none=True,  
            exclude_unset=True,  
            by_alias=True,  
        ) #serializing the model to json. exclude_none, exclude_unset and by_alias are important!
    )
```

Lastly, non-returning `DELETE` requests can simply be specified like this:
```python
def delete_database_user(self, database_id: str, user_id: str):  
    self._http.request("DELETE", f"databases/{database_id}/users/{user_id}")
```