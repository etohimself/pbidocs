# Power BI Docs

**Power BI Docs** is a single‐page HTML documentation generator for Power BI. It produces makedocs/jsdoc‑style documentation from metadata and allows you to export a read‐only, self‐contained HTML file or import and edit existing documents.

--- 

## Key Features

- **Interactive Navigation:**  
  Easily browse your documentation using a dynamic sidebar that supports hierarchical structures. Expand and collapse categories and items with a simple click.

- **Live Editing:**  
  Update item descriptions and DAX formulas in an intuitive modal editor. Benefit from enhanced editing with real‐time syntax highlighting and precise caret alignment.

- **Auto‑Linking References:**  
  Automatically transform references to measures, tables, and dimensions into clickable links. Navigate seamlessly to related sections without manual setup.

- **Slash Commands in Editor:**  
  Type `/` in the description editor to quickly insert references to existing items using convenient slash commands.

- **Export to Read‑Only HTML:**  
  Generate a self‐contained, read‐only HTML file that preserves navigation and search functionality while disabling editing features—ideal for sharing with stakeholders or publishing online.

- **Import Existing Documents:**  
  Load pre‐existing documentation, edit it, and re‑export as needed.

- **Enhanced Syntax Highlighting:**  
  Enjoy built‐in Prism‑based syntax highlighting for DAX, making your formulas visually appealing and easier to understand.

--- 

## Getting Started

### Installation

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/etohimself/pbidocs
   cd pbidocs
   ``` 

2. **Locate Your SSAS Port and Database ID:**

   Open your PBIX file in Power BI Desktop. Navigate to:  
   `%LocalAppData%\Microsoft\Power BI Desktop\AnalysisServicesWorkspaces`  
   and note the `msmdsrv.port` and `uuid.db` files.

3. **Generate Metadata with Power Query:**

   Open a new Power BI report and launch Power Query. Run the following M‑code (after updating the port and database ID accordingly):

   ```m
   let
       metadata = AnalysisServices.Databases(
           "127.0.0.1:58860", 
           [ Implementation = "2.0", TypedMeasureColumns = true ]
       ){[Name="1ca2eb63-e90e-42ee-9ece-c84d78539f45"]}[Data]{[Id="Model"]}[Data]{[Id="Model"]}[Data],
   
       RawDimensions = Cube.Dimensions(metadata),
   
       DimensionColumnNames = Table.TransformColumns(
           RawDimensions,
           {{"Data", each Table.ColumnNames(_)}}
       ),
       
       DimensionColumnsExpanded = Table.ExpandListColumn(DimensionColumnNames, "Data"),
   
       DimensionParentRemoved = Table.AddColumn(
           DimensionColumnsExpanded,
           "CleanData",
           each Text.Replace([Data], [Id] & ".", ""),
           type text
       ),
   
       DimensionFirstOccurance = Table.TransformColumns(DimensionParentRemoved, {{"CleanData", each List.First(Text.Split(_, "."))}}),
   
       DimensionNoBrackers = Table.TransformColumns(DimensionFirstOccurance, {{"CleanData", each Text.Replace(Text.Replace(_, "[", ""), "]", "")}}),
   
       DimensionRenamed = Table.RenameColumns(
           Table.SelectColumns(DimensionNoBrackers, {"Name", "CleanData"}),
           {{"Name", "Parent"}, {"CleanData", "Name"}}
       ),
   
       DimensionTypeAdded = Table.AddColumn(DimensionRenamed, "Type", each "Dimension"),
   
       RawMeasures = Cube.Measures(metadata),
   
       MeasureNameOnly = Table.SelectColumns(RawMeasures, {"Name"}),
   
       MeasureParentAdded = Table.AddColumn(MeasureNameOnly, "Parent", each "Measures"),
   
       MeasureTypeAdded = Table.AddColumn(MeasureParentAdded, "Type", each "Measure"),
   
       RawTables = Cube.Dimensions(metadata),
   
       TablesNameOnly = Table.SelectColumns(RawTables, {"Name"}),
   
       TablesParentAdded = Table.AddColumn(TablesNameOnly, "Parent", each "Tables"),
   
       TablesTypeAdded = Table.AddColumn(TablesParentAdded, "Type", each "Table"),
   
       FinalFormat = Table.Combine({MeasureTypeAdded, DimensionTypeAdded, TablesTypeAdded})
   
   in
       FinalFormat
   ``` 

4. **Open the Editor:**

   Open the cloned folder and launch the `index.html` file in your browser. Copy and paste the metadata (in tab‑separated format) generated via Power Query. For example, your data might look like this:

   ```tsv
   Name	Parent	Type
   Agent Handled Calls	Measures	Measure
   Agent Avg. Calls	Measures	Measure
   Date	Dim_Date	Dimension
   Week Year	Dim_Date	Dimension
   Agent	Fact_TestData	Dimension
   Handled Calls	Fact_TestData	Dimension
   Dim_Date	Tables	Table
   Fact_TestData	Tables	Table
   ``` 

### Usage

1. **Create Documentation from Data:**

   - On the welcome screen, click **"Create from Data"**.
   - Paste your raw, tab‑separated data into the provided modal.

2. **Import Existing Documentation:**

   - Click **"Import from Existing Documentation"** to load a previously exported HTML file containing your documentation.

3. **Edit Documentation:**

   - Select an item from the sidebar to view its details.
   - Click the **"Edit"** button to open the modal editor.
   - Update the item's name, description, or formula using the enhanced editors. The description editor supports slash commands for inserting references, and the formula editor provides real‐time DAX syntax highlighting.

4. **Export Documentation:**

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
