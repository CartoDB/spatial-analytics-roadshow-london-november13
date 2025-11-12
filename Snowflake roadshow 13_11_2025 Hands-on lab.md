# **Get Hands On: Analyze, Visualize & Reason in CARTO**

In this workshop, we’ll be learning how to transform a simple map and workflow into a decision-making force\! 

To get started, you’ll need to login to the CARTO account that we’ve set up for you at [app.carto.com](http://app.carto.com). You can find the details for this at your desk \- it will look something like:

* Username: snowflakeroadshow+user1@cartodb.com  
* Password: Carto123\!

**Prefer to just watch? If you login, you can explore the final map we create [here](https://clausa.app.carto.com/viewer/8d4654c2-ed71-4d0c-8d3c-8b211b5c52eb).** 

<img width="1600" height="848" alt="CARTO Workspace" src="https://github.com/user-attachments/assets/e34e144e-7a29-4aaf-a9d6-c142072c1887" />



*The CARTO Workspace*

---

# **Part 1: Building a map** {#part-1:-building-a-map}

1. In the **Maps** tab, locate the map **London collisions map\_TEMPLATE**.   
2. Click the three dots next to it to duplicate a copy. Once opened, rename it from “TEMPLATE” to your own name.   
3. Open the map, and start exploring\! Try restyling your layer or adding some widgets\!

<img width="1600" height="861" alt="Builder map viewer mode" src="https://github.com/user-attachments/assets/a12e4ab0-0ee3-4839-a186-21db47557416" />


---

# **Part 2: Adding an agent** {#part-2:-adding-an-agent}

Now we want to turn our dashboard from an interactive map into a full reasoning experience\!

1. First, we need to add an agent \- at the top-left of the window, open the **AI Agents** panel ![][image3].  
2. Click **Create Agent**.  
3. This will open the Agent Configuration panel \- let’s set up a simple agent by pasting the below information into the relevant sections.

**Use case:**

| Analyzing risk based on traffic collisions in London. |
| :---- |

**Instructions:**

```markdown
## General behavior
- Provide clear, contextual and concise responses, using bullets and tables where possible – rather than paragraphs of text.  
- Only provide one message per user request, not per internal step. Do not repeat yourself.  
- Provide suggestions for follow-up analysis and prompts in the context of risk analysis and transport planning.  
- Prioritize doing filtering and analysis with widgets & parameters over running SQL queries.  
- When running any analysis that involves filtering data to a time period, you should always use the parameters to update the map so it just shows the relevant data.  
- Run through task steps sequentially, pausing for user input when specified.  
- The collision data is available from 01/01/2000 to 31/12/2024. If the user tries to specify dates outside this time (e.g., all of 2025) you should make them aware of the data limitations.  
- Don't report an error until you have exhausted all options.  

## Data sources
You may use the following data sources for your analysis:  
1. `"SNOWFLAKE_ROADSHOW_25"."PUBLIC"."LONDON_COLLISIONS"` – point locations of collisions in London  
2. `"SNOWFLAKE_ROADSHOW_25"."PUBLIC"."LONDON_WARDS"` – wards. Fields: `WD21CD = unique ID`, `WD21NM = ward name`.  

## Task 1: collisions by location
- If asked to calculate the number of collisions by area (e.g., within a ward), `/execute_query` using the following SQL as a template:

```sql
SELECT
  w.WD21CD,
  w.WD21NM,
  ANY_VALUE(w.GEOM) AS GEOM,
  COUNT(c.COLLISION_INDEX) AS collisions_count
FROM "SNOWFLAKE_ROADSHOW_25"."PUBLIC"."LONDON_WARDS" w
LEFT JOIN "SNOWFLAKE_ROADSHOW_25"."PUBLIC"."LONDON_COLLISIONS" c
  ON ST_INTERSECTS(w.GEOM, c.GEOM)   -- point-in-polygon
GROUP BY
  w.WD21CD,
  w.WD21NM
ORDER BY
  collisions_count DESC;
```

- `/add_source` and `/add_layer` to the map showing the wards with a sequential fill showing number of collisions.  
- Do not add a tooltip.  
- Turn off the collisions (locations) layer.
```

**Query sources for insights:** enabled

**Model:** gemini-2.5-pro

**Welcome message:**

| Your AI Agent for understanding collision risk in London. |
| :---- |

**Conversation starters: (Make sure you select \+ to add this)**

| Summarize collisions by ward |
| :---- |

4. Hit save\!   
5. Back in your map, you can see the agent is ready to use\! Try clicking the starter prompt, or asking it a question like “Describe year-on-year trends of serious and fatal collisions” or “Show me collisions involving floods.” 

<img width="1600" height="862" alt="using AI Agent" src="https://github.com/user-attachments/assets/04a12dcd-2816-4a25-97b8-bb5d9c39b63a" />


Now try running the “Which 5 wards had the most collisions?” prompt again… it should respond by asking you to specify a time period first.

Congratulations \- you’ve just created your first AI Agent\! Now, let’s explore how we can connect this to external tools. 

---

# **Part 3: Building an MCP tool** {#part-3:-building-an-mcp-tool}

**💡Get stuck? [Here’s](https://clausa.app.carto.com/workflows/53932c18-7c6d-4508-9998-17e941b0cf7d) one we created earlier for you to explore.**

Now we’re going to create a simple MCP tool that calculates collision hotspots, allowing our map user to define the time period for the hotspots. 

1. First, in the main CARTO workspace ([app.carto.com](http://app.carto.com)) open the **Workflows** tab.  
2. Find the workflow **Collision hotspots\_TEMPLATE**. Just like we did with the map, click the three dots here to duplicate the workflow.  
3. Once opened, rename it from “TEMPLATE” to your own name. You should be looking at something like this…

<img width="1600" height="863" alt="workflow 1" src="https://github.com/user-attachments/assets/5c422bc3-78fe-4825-97dc-fd38a7c9956f" />


Now, let’s turn it into a tool that connects to our Agent\! 

4. First, let’s set some variables that allow the user to customize the workflow through natural language. Open the **Variables** tab (next to Run) and add the following variables:

| Order | Name | Type | Default value | MCP Tool |      API |
| :---- | :---- | :---- | :---- | ----- | ----- |
| 1\. | start\_date | String | 2000-01-01 |  ✅  |  |
| 2\. | end\_date | String | 2024-12-31 | :✅ |  |

5. Now, we need some way to pass these variables into our workflow. On the left side of the screen, switch from Sources to **Components**. Search for and add a **Where** component to the canvas, linking it to the LONDON\_COLLISIONS table and H3 from Geopoint.  
6. Add the below SQL to the Where component.

```sql
geom IS NOT NULL
  AND DATE(COLLISION_DATE) BETWEEN
      TO_DATE(@start_date) AND TO_DATE(@end_date)
```

| :---- |

 

7. Run the workflow just to check the variables are being passed correctly.   
8. Finally, add an **MCP Tool Output** component to the end of your workflow and set the output type to **Sync** \- this is how the workflow knows to send its output back to our map. 

Your workflow should now look like the below, with the sections you’ve added highlighted in yellow.

<img width="1600" height="863" alt="Workflow MCP tool" src="https://github.com/user-attachments/assets/acab7e72-a66f-47e4-b25e-dd4842c9b6e0" />


9. Now, open Workflow settings (the three dots at the top right of the screen) and open the **MCP Tools** window.   
10. Copy the below text into the relevant boxes:

**Tool description:**

| Calculates statistically significant collision hotspots in London on a H3 grid. This tool requires a start and end date, between 2000-01-01 and 2024-12-31. Always outputs H3 cells in a column named H3. |
| :---- |

**Inputs \- start date**

| Start date to filter collisions  |
| :---- |

**Inputs \- end date**

| End date to filter collisions  |
| :---- |

**Output**

| A table of H3 cells with associated statistics (GI score, p-value, count),  representing statistically significant collision hotspots within the specified time period.  All results are guaranteed to use an H3 grid. |
| :---- |

11. Once complete, click **Enable as MCP tool** \- and your workflow is ready to use in an Agent\! 

<img width="1600" height="863" alt="MCP tool" src="https://github.com/user-attachments/assets/2fd0f773-e294-457b-8dce-773eeb585df4" />


Now you’re ready to use your MCP tool in your agent\!

---

# **Part 4: Putting it all together** {#part-4:-putting-it-all-together}

**💡Get stuck? [Here’s](https://clausa.app.carto.com/viewer/02b7d3b5-40c5-4f13-b3e7-5da851f5a14d) one we created earlier for you to explore.**

In this final section, we’re going to add our MCP tool to our Agent.

1. Refresh your Builder map, then open the **Agent Configuration** window again.   
2. Underneath your Instructions, you should see a section called **MCP tools**. Click **\+ Add tools** \- you should see your MCP tool ready to use\!  
3. Click **\+** to add this tool to your Agent.

❗If you get stuck, you should find a tool called “Collision hotspots example\_final” which we created which you can use.

4. Now, let’s add a second “task” to our agent to define how this tool should be used. Copy the below into the bottom of your instructions. Instead of your tool, type “/” and you should see a list of tools available to use \- select your MCP tool here.

```markdown
## Task 2: Where are there collision hotspots?
1. Confirm the time period with the user and use the SQL parameters to update the map.  
- Tell the user:  
"I will run the required Workflow as an MCP tool – this should take about a minute. You can continue exploring the map during this time."

2. Run the /yourtool tool using the selected time period.  
- The parameters must be specified in the exact order: `"queryParameters":{"start_date":"", "end_date":""}`  
- If you encounter the `WRONG_GRID_TYPE_EXCEPTION` error, do not report it – keep attempting to rerun the tool.

3. When the workflow completes, you must use the `/add_source` tool with the following parameters to add the results as a new H3 layer.  
Do not use the `/add_source_from_workflows` tool.  

```
type = 'query'  
sql = SELECT * FROM <workflow_output_table>  
geoColumn = 'H3'  
```

After adding the source, add it to the map as a H3 layer with a red fill and a dark red stroke.  
Do not add tooltips.  

4. Ensure the collision data is filtered to the selected time period using the SQL parameters.
```


5. To make things even easier for your user, add a second conversation starter “Run collision hotspot analysis.” Save your agent and return to the map.   
6. Now, try testing out your new MCP tool by selecting your new conversation starter\!

<img width="1600" height="861" alt="Final agent" src="https://github.com/user-attachments/assets/9f4776d4-5d48-4092-8e46-41dc6de78d76" />

