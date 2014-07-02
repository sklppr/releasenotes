# Bamboo Release Notes Generator

## Why?

Generating release notes by hand and listing everything that has changed can be cumbersome and time consuming.

This generator script generates release notes by fetching JIRA issues associated with your builds, grouping them by sprint or build number and indenting sub-tasks and blocking issues underneath their parent issues.

Additionally, you can:

- add your own message,
- include the branch name,
- restrict the number of issues and builds,
- filter issues by type or status,
- change how issues are grouped,
- change how issues are displayed and
- change the release notes file name.

## Installation

### Installing the release notes generator

**Prerequisites:** JIRA 6.2, Bamboo 5.5 and Ruby 2.0 (other versions might work but weren’t tested).

The generator script is self-contained. Just place it somewhere and make it executable: `chmod +x releasenotes`

Add a new task of type “Command” to your build plan and enter a description (e.g. “Generate Release Notes”).  
Click “Add new executable”, enter a name (e.g. “Release Notes Generator”) and specify the absolute path to the executable.

Don’t forget to also define an artifact for the release notes file (default file name: `releasenotes.md`, customizable).

### Configuring the release notes generator

There are multiple ways to set environment variables:

1. Create a **global variable** or **build plan variable** in Bamboo. In this case, leave off the `bamboo_` prefix since Bamboo will add that automatically.
2. Specify an **environment variable** for the Bamboo build task using the following syntax: `bamboo_Plan_Key="TEST-PLAN"`
3. Place a **`releasenotes.env` file** in the same location as the script. The file is using YAML syntax: `bamboo_Plan_Key: TEST-PLAN`

In general, environment variables set on a build task will overwrite build plan variables which will overwrite global variables. Global or build plan variables are useful because they can be overridden using a custom build. However, some variables *have* to be set on the build task in order to use variable substitution.

The variables from the `releasenotes.env` file won’t overwrite environment variables that are already set. Therefore, the file can be used for global configuration, storing sensitive information (e.g. passwords) or providing defaults that users can override in Bamboo.

#### Servers and authentication

Specify server URLs including procotol and, if applicable, port:

- `bamboo_JIRA_Server` – JIRA URL (e.g. `https://jira.example.com`)
- `bamboo_Bamboo_Server` – Bamboo URL (e.g. `http://bamboo.example.com:8085`)

Either specify login data either for JIRA and Bamboo separately:

- `bamboo_JIRA_User` – JIRA username
- `bamboo_JIRA_Password` – JIRA password
- `bamboo_Bamboo_User` – Bamboo username
- `bamboo_Bamboo_Password` – Bamboo password

… or a single user for both of them:

- `bamboo_Atlassian_User` – JIRA + Bamboo username
- `bamboo_Atlassian_Password` – JIRA + Bamboo password

#### Build variables

Either specify build plan, stage and number separately:

- `bamboo_Build_Plan` – Key of build plan (e.g. `TEST-PLAN`)
- `bamboo_Build_Stage`– Key of current build stage (e.g. `DELIVER`), must be set on build task: `bamboo_Build_Stage="${bamboo.shortJobKey}"`
- `bamboo_Build_Number` – Number of current build (e.g. `42`), must be set on build task: `bamboo_Build_Number="${bamboo.buildNumber}"`

… or provide the build result key and optionally a pattern to parse it:

- `bamboo_Build_Result` – Key of current build result (e.g. `TEST-PLAN-DELIVER-42`), must be set on build task: `bamboo_Build_Result="${bamboo.buildResultKey}"`
- `bamboo_Build_Result_Pattern` – Regular expression to parse build plan, stage and number from the build result key. The expression must provide the named capture groups `plan`, `stage` and `number`. Default: `^(?<plan>[\w-]+)-(?<stage>\w+)-(?<number>\d+)$` which allows dashes in the build plan but not the build stage (e.g. `TEST-PLAN-DELIVER-42`).

Finally, specify the custom field where JIRA Agile stores its sprint data:

- `bamboo_Sprint_Field` – Name of custom field containing JIRA Agile sprint, e.g. `customfield_12345`

#### Example setup

1. Add `bamboo_Build_Result="${bamboo.buildResultKey}"` to the build task.
2. Specify `bamboo_Sprint_Field`, `bamboo_Bamboo_Server`, `bamboo_JIRA_Server`, `bamboo_Atlassian_User` and `bamboo_Atlassian_Password` on the build plan or in the environment file.

## Usage

### Generating release notes from JIRA issues

Release notes are generated from JIRA issues that were linked to Bamboo builds. Issues can be linked to builds in two ways:

- Automatically by using “[smart commits](https://confluence.atlassian.com/display/AOD/Processing+JIRA+issues+with+commit+messages)”. Simply mention the JIRA issue key in the commit message (e.g. `TEST-PROJECT-123 Fixed some really bad bug.`) and the issue will be linked to the next build you run in Bamboo.
- Manually when running a build in Bamboo. This is especially useful when the stage in which release notes are generated is set to “manual”. **Important:** Make sure to link the issues to the *build* itself, not a *build stage*.

The script will fetch all JIRA issues linked to any build in the build plan, including the current build.

### Adding a custom message

You can add a custom message to your release notes by providing a variable called `bamboo_Release_Notes_Message`.  
This message will appear between heading and issues and can also contain Markdown.

It’s usually best to define this variable on your build plan. You can then use it in a couple of ways:

- Enter a permanent message and leave it unchanged.
- Leave it empty and later add a message using a customised build.
- Enter a default message and override it using a customised build.

**How to run a customised build:** Click “Run” > “Run customised…”, then click “Override a variable” and select “Release_Notes_Message”.

### Including the branch name

You can include the name of the current branch in your release notes by adding an environment variable named `bamboo_Branch_Name` to your build task and assigning Bamboo’s `bamboo.repository.*.branch` global variable to it:

- Git: `bamboo_Branch_Name="${bamboo.repository.git.branch}"`
- Mercurial: `bamboo_Branch_Name="${bamboo.repository.hg.branch}"`
- Subversion: `bamboo_Branch_Name="${bamboo.repository.svn.branch}"`

### Restricting the number of issues and builds

For API requests, both to Bamboo and JIRA, a maximum of 1000 results is requested instead of the default values.  
You can override this number using the following variables:

- `bamboo_Max_Builds` – maximum number of builds requested from Bamboo (default: 1000)
- `bamboo_Max_Issues` – maximum number of issues requested from JIRA (default: 1000)

### Filtering issues by type or status

By default, all issues linked to a build plan are displayed. To change this, provide the following variables:

- `bamboo_Release_Notes_Issue_Type` – allowed issue types (e.g. Epic, User Story, New Feature, Improvement, Bug, Impediment, Task, Sub-task)
- `bamboo_Release_Notes_Issue_Status` – allowed issue status (e.g. Open, To Do, In Progress, Blocked, Closed, Done)

The value is expected to be a comma-separated list of allowed issue types or status, i.e. a whitelist.

**Important:** Issue types and status are not case sensitive but have to be spelled exactly as defined in JIRA (including spaces).

### Changing how issues are grouped

You can change how issues a group by providing a variable called `bamboo_Release_Notes_Mode` and specifying the display mode.

Available display modes are:

- `sprint` (default) – Issues are grouped by sprint and sort descending (newest sprint first).
- `build` – Issues are grouped by build number and sort descending (newest build first).

### Changing how issues are displayed

By default, several pieces of information are displayed alongside each issue. You can control this by providing a variable called `bamboo_Release_Notes_Issue_Info` with a comma-separated list of infos to display.

Available pieces of information are:

- `type` – issue type (e.g. User Story, New Feature, Bug)
- `status` – issue status (e.g. Closed, In Progress)
- `build` – number of the last build this issue was associated with

### Changing the file name

You can change the name of the release notes file by providing a variable called `bamboo_Release_Notes_File`. The default file name is `releasenotes.md`.
