Autopsy is an open-source an [[Digital Forensics]] platform. 
[**The official description**](https://www.autopsy.com/): "_Autopsy is the premier open source forensics platform which is fast, easy-to-use, and capable of analysing all types of mobile devices and digital media. Its plug-in architecture enables extensibility from community-developed or custom-built modules. Autopsy evolves to meet the needs of hundreds of thousands of professionals in law enforcement, national security, litigation support, and corporate investigation._"

Basic workflow:

1. Create/open the case for the data source you will investigate
2. Select the data source you wish to analyse
3. Configure the ingest modules to extract specific artefacts from the data source
4. Review the artefacts extracted by the ingest modules
5. Create the report

**Case Analysis | Open an Existing Case**

The Autopsy can also open prebuilt case files. Note that supported data sources are discussed in the next task. This part aims to show how to create/open case files with Autopsy.

**Note:** Autopsy case files have a ".aut" file extension.

**Data Sources**
Supported Disk Image Formats:
- **Raw Single** (For example: *.img, *.dd, *.raw, *.bin)
- **Raw Split** (For example: *.001, *.002, *.aa, *.ab, etc)
- **EnCase** (For example: *.e01, *.e02, etc)
- **Virtual Machines** (For example: *.vmdk, *.vhd)

**Ingest Modules**

Essentially **Ingest Modules** are Autopsy plug-ins. Each Ingest Module is designed to analyse and retrieve specific data from the drive. You can configure Autopsy to run specific modules during the source-adding stage or later by choosing the target data source available on the dashboard. By default, the Ingest Modules are configured to run on All Files, Directories, and Unallocated Space.

**User Interface**
**Tree Viewer**
The **Tree Viewer** has **five top-level nodes**:

- **Data Sources** - all the data will be organised as you would typically see it in a normal Windows File Explorer. 
- **Views** - files will be organised based on file types, MIME types, file size, etc. 
- **Results** - as mentioned earlier, this is where the results from Ingest Modules will appear. 
- **Tags** - will display files and/or results that have been tagged (read more about tagging [here](http://sleuthkit.org/autopsy/docs/user-docs/4.12.0/tagging_page.html)).
- **Reports** - will display reports either generated by modules or the analyst (read more about reporting [here](http://sleuthkit.org/autopsy/docs/user-docs/4.12.0/reporting_page.html)).

**Result Viewer**

**Note**: Don't confuse the Results node (from the Tree Viewer) with the Result Viewer. 

When a volume, file, folder, etc., is selected from the Tree Viewer, additional information about the selected item is displayed in the Result Viewer. For example, the Sample case's data source is selected, and now additional information is visible in the Results Viewer.

**Contents Viewer**

From the Table tab in the Result Viewer, if you click any folder/file, additional information is displayed in the Contents Viewer pane.

- **S** = **Score**

The **Score** will show a red exclamation point for a folder/file marked/tagged as notable and a yellow triangle pointing downward for a folder/file marked/tagged as suspicious. These items can be marked/tagged by an Ingest Module or the analyst.

- **C** = **Comment**

If a yellow page is visible in the Comment column, it will indicate that there is a comment for the folder/file. 

- **O** = **Occurrence** 

In a nutshell, this column will indicate how many times this file/folder has been seen in past cases (this will require the [Central Repository](http://sleuthkit.org/autopsy/docs/user-docs/4.12.0/central_repo_page.html))

Generate Report

You can create a report of your findings in multiple formats, enabling you to create data sheets for your investigation case. The report provides all information listed under the "Result Viewer" pane. Reports can help you to re-investigate findings after finishing the live investigation. **However, reports don't have additional search options, so you must manually find artefacts for the event of interest.**

**Tip:** The Autopsy tool can be heavy for systems with low resources. Therefore completing an investigation with Autopsy on low resources can be slow and painful. Especially browsing long results might end up with a system freeze. You can avoid that situation by using reports. You can use the tool for parsing the data and generating the report, then continue to analyse through the generated report without a need for Autopsy. Note that it is always easier to conduct and manage an investigation with the GUI.

**Visualisation Tools**

**The Timeline tool is composed of three areas:**

1. **Filters:** Narrow the events displayed based on the filter criteria
2. **Events:** The events are displayed here based on the **View Mode**
3. **Files/Contents:** Additional information on the event(s) is displayed in this area

**There are three view modes:**

1. **Counts:** The number of events is displayed in a bar chart view
2. **Details:** Information on events is displayed, but they are clustered and collapsed, so the UI is not overloaded
3. **List:** The events are displayed in a table view