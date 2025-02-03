# Power BI Docs

**Power BI Docs** is a single‐page HTML documentation generator for Power BI. It produces makedocs/jsdoc‑style documentation from metadata and allows you to export a read‐only, self‐contained HTML file or import and edit existing documents.

--- 
## Installation
**Clone the Repository:**

   ```bash
   git clone https://github.com/etohimself/pbidocs
   cd pbidocs
   ```
## Document Your Reports with Proper Syntax
--- 
1. **Use Description Syntax In Your DAX and M-Code :**
    Write your measure and calculated expressions as follows :
    ```dax
     /**
     Example description for a DAX measure. 
     Link to a measure : [Some Measure Name]
     Link to a table's dimension : Fact_Exampletable[Example Dimension]
     Link to a table : Fact_ExampleTable
      **/
     CALCULATE([Some Measure], ALL())
     ```

    Write your M-Code as follows :
    ```dax
     /** 
     Example description for Table definition with M-Code.
     Link to a measure : [Some Measure Name]
     Link to a table's dimension : Fact_Exampletable[Example Dimension]
     Link to a table : Fact_ExampleTable
     **/
     let
        Source = ...
     in 
        Source
     ```

## Extract Metadata From Your Power BI Report
--- 
1. **Locate Your SSAS Port and Database ID:**

   Open your PBIX file in Power BI Desktop. Navigate to:  
   `%LocalAppData%\Microsoft\Power BI Desktop\AnalysisServicesWorkspaces`  
   and note the `msmdsrv.port` and `uuid.db` files.

2. **Generate Metadata with Power Query:**

   Open a new Power BI report and launch Power Query. Run the following M‑code (after updating the port and database ID accordingly):

   ```m
   let
    // Set your connection parameters
    server = "127.0.0.1:__YOUR_SSASS_PORT_HERE__",
    databaseID = "__YOUR_DATABASE_UUID_HERE__,
    
    // Connect to your SSAS Tabular model
    Source = AnalysisServices.Databases(server, [ Implementation = "2.0", TypedMeasureColumns = true ]),
    Database = Source{[Name = databaseID]}[Data],    
    DebugResults = Value.NativeQuery(Database, "SELECT [ExplicitName], [TableID] FROM $SYSTEM.TMSCHEMA_COLUMNS"),

    // --- Query 1: Retrieve Measures using SQL-like syntax
    MeasuresQuery = "
        SELECT 
            [Name] AS [Name],
            'Measures' AS [Parent],
            'Measure' AS [Type],
            [Expression] AS [Expression]
        FROM $SYSTEM.TMSCHEMA_MEASURES
    ",
    MeasuresRaw = Value.NativeQuery(Database, MeasuresQuery),
    Measures = Table.TransformColumns(MeasuresRaw, {{"Expression", each if _ = null then "" else _}}),
    
    // --- Query 2: Retrieve Calculated Tables using SQL-like syntax
    TablesQuery = "
        SELECT 
            [Name] AS [Name],
            'Tables' AS [Parent],
            'Table' AS [Type],
            [TableID],
            [QueryDefinition] AS [Expression]
        FROM $SYSTEM.TMSCHEMA_PARTITIONS
        WHERE [Type] = '4'
    ",
    TablesRaw = Value.NativeQuery(Database, TablesQuery),
    Tables = Table.RemoveColumns(Table.TransformColumns(TablesRaw, {{"Expression", each if _ = null then "" else _}}), {"TableID"}),
    
    // --- Query 3: Retrieve Columns (for dimensions and calculated columns) using SQL-like syntax
    ColumnsQuery = "
    SELECT 
        [ExplicitName] AS [Name],
        [TableID],
        'Dimension' AS [Type],
        [Expression] AS [Expression]
    FROM $SYSTEM.TMSCHEMA_COLUMNS
    ",
    ColumnsRaw = Value.NativeQuery(Database, ColumnsQuery),
    ColumnsJoinStep1 = Table.NestedJoin(ColumnsRaw, {"TableID"}, TablesRaw, {"TableID"}, "nestedTableData", JoinKind.LeftOuter),
    ColumnsJoinStep2 = Table.ExpandTableColumn(ColumnsJoinStep1, "nestedTableData", {"Name"}, {"Parent"} ),
    Columns = Table.RemoveColumns(Table.TransformColumns(ColumnsJoinStep2, {{"Expression", each if _ = null then "" else _}}), {"TableID"}),
    
    // Combine the results into one final table
    FinalFormat = Table.Combine({Measures, Tables, Columns}),
    #"Filtered Rows" = Table.SelectRows(FinalFormat, each ([Parent] <> null)),
    #"Filtered Rows1" = Table.SelectRows(#"Filtered Rows", each not Text.Contains([Name], "RowNumber-")) 
    in
    #"Filtered Rows1"
   ```
## Generate Documentation from Metadata
--- 
1. **Open the Editor:**
   Open the cloned folder and launch the `index.html` file in your browser. Copy and paste the metadata (in tab‑separated format) generated via Power Query. For example, your data might look like this:

   ```tsv
   Name	Parent	Type Expression
   Agent Handled Calls	Measures	Measure	CALCULATE([Measure], ALL())
   Agent Avg. Calls	Measures	Measure	CALCULATE([Measure], ALL())
   Date	Dim_Date	Dimension	CALCULATE([Measure], ALL())
   Week Year	Dim_Date	Dimension	CALCULATE([Measure], ALL())
   Agent	Fact_TestData	Dimension	CALCULATE([Measure], ALL())
   Handled Calls	Fact_TestData	Dimension	CALCULATE([Measure], ALL())
   Dim_Date	Tables	Table	CALCULATE([Measure], ALL())
   Fact_TestData	Tables	Table	CALCULATE([Measure], ALL())
   ```

2. **Create Documentation from Data:**

   - On the welcome screen, click **"Create from Data"**.
   - Paste your raw, tab‑separated data into the provided modal.

3. **Import Existing Documentation:**

   - Click **"Import from Existing Documentation"** to load a previously exported HTML file containing your documentation.

4. **Edit Documentation:**

   - Select an item from the sidebar to view its details.
   - Click the **"Edit"** button to open the modal editor.
   - Update the item's name, description, or formula using the enhanced editors. The description editor supports slash commands for inserting references, and the formula editor provides real‐time DAX syntax highlighting.

5. **Export Documentation:**

   - Once your documentation is complete, click **"Export Docs"**.
   - A read‐only HTML file will be generated that retains navigation and search functionality while disabling editing features.

--- 

## License

This project is licensed under the [MIT License](LICENSE).

--- 

## Credits

This project leverages [Prism.js](https://prismjs.com/) for enhanced syntax highlighting. We gratefully acknowledge the hard work and dedication of the Prism.js team and its contributors. Their innovative efforts have greatly enriched our project's functionality and user experience.


## Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request for improvements, bug fixes, or new features. A future enhancement will include change detection with an automatically generated changelog displayed on the main page.

--- 

## Contact

For any questions or feedback, please reach out at [ertugrulcore@gmail.com](mailto:ertugrulcore@gmail.com).

--- 

Enjoy your **Power BI Docs**!
