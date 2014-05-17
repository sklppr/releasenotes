# Bamboo Release Notes Generator

## Installation

The script is self-contained. Just place it somewhere and make it executable: `chmod +x releasenotes`

Now you can add it to your Bamboo build plan:

1. Add a new “Command” build task. Enter a description (e.g. “Generate Release Notes”).
3. Click “Add new executable”, enter a name (e.g. “Release Notes Generator”) and the absolute path to the script.

## Configuration

The generator expects the following environment variables to be set:

- `bamboo_Bamboo_Server` – Bamboo URL including `http(s)://`
- `bamboo_Bamboo_User` – Bamboo username
- `bamboo_Bamboo_Password` – Bamboo password
- `bamboo_JIRA_Server` – JIRA URL including `http(s)://`
- `bamboo_JIRA_User` – JIRA username
- `bamboo_JIRA_Password` – JIRA password
- `bamboo_Plan_Key` – Key of build plan in Bamboo (*not key of build stage or build result*), e.g. `TEST-PLAN`

Optionally, the following environment variables can be set:

- `bamboo_Release_Notes` – Custom text to be displayed at top of release notes, default: empty
- `bamboo_Release_Notes_File` – File name for release notes, default: `releasenotes.md`
- `bamboo_Issue_Types` – Comma-separated list of issue types to display in desired order, default: `Epic, Story, New Feature, Improvement, Bug, Task, Subtask`

There are multiple ways to set environment variables:

1. Create a **build plan variable** in Bamboo. In this case, leave off the `bamboo_` prefix since Bamboo will add that automatically.
2. Specify an **environment variable** for the Bamboo build task using the following syntax: `bamboo_Plan_Key="TEST-PLAN"`
3. Place a **`releasenotes.env` file** in the same location as the script. The file is using YAML syntax: `bamboo_Plan_Key: TEST-PLAN`

**Important:** Environment variables set on a build task will overwrite build plan variables. However, the variables from the `.env` file won’t overwrite environment variables that are already set, so the file can be used to provide default values.