# WebUntis Timetable to ICS

[![Update Calendar and Deploy to GitHub Pages](https://github.com/Chaos02/WebUntisTimeTableToIcs/actions/workflows/GeneratePage.yml/badge.svg)](https://github.com/Chaos02/WebUntisTimeTableToIcs/actions/workflows/GeneratePage.yml) Last Push: [![Update Calendar and Deploy to GitHub Pages](https://github.com/Chaos02/WebUntisTimeTableToIcs/actions/workflows/GeneratePage.yml/badge.svg?event=push)](https://github.com/Chaos02/WebUntisTimeTableToIcs/actions/workflows/GeneratePage.yml) [![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2FChaos02%2FWebUntisTimeTableToIcs%2Factions%2Fworkflows%2F119631616%2Fruns%3Fstatus%3Dcompleted%26per_page%3D1&query=%24.workflow_runs%5B0%5D.run_started_at&label=Last%20Successful%20Run%3A)](https://github.com/Chaos02/WebUntisTimeTableToIcs/actions/workflows/GeneratePage.yml)


Scrapes the official API the website uses and processes the entries into a (or multiple) subscribable ICS files.

## Usage

1. Fork this repository
2. Enable GitHub Pages with workflow as source.
3. Store the environment variables `BASE_URL`, `ELEMENT_ID`, `OVERRIDE_SUMMARIES`, `COOKIE` and `TENANT_ID` in the repos **secrets**:
    - BASE_URL: `#####.webuntis.com` (Your officially given WebUntis website, REQUIRES)
    - COOKIE: `##################` (Get by opening official page with dev tools, right click the request (see below) and `Copy as PowerShell`)
    - ELEMENT_ID: `####` (Course ID, 4 digits long, REQUIRED)
    - OVERRIDE_SUMMARIES: `@{"GK" = "GK, Gemeinschaftskunde";"LBTL1" = "EL, Elektrotechnik";...}` (opt.)
    - CULTURE: `de-DE` (Adjust to your preference, any [.NET recognized language tag](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-lcid/a9eac961-e77d-41a6-90a5-ce1a8b0cdb9c)) (opt.)
    - TENANT_ID: `#######` (School ID under this instance (REQUIRED))
5. Adjust cronjob in ./.github/workflows/GeneratePage.yml to your needs.
6. Adjust step `Run PowerShell Script and Capture Output` if you don't want to append to a previous ICS or override course names

<details>
<summary>How to get Parameters?</summary>

### Acquiring Parameters

Open your webuntis instance in a web browser with development tools, navigate to your timetable.
Copy and open the share-able URL: 
![Screenshot URL Button](https://i.imgur.com/EeZ8WyX.png)
( -> `https://tipo.webuntis.com/WebUntis?school=*************#/basic/timetablePublic/class?date=***********&entityId=<ELEMENT_ID>`)

Then, locate the Web Request starting with `entries` and use the **redacted highlighted Values** for the Secrets:
![Screenshot Dev Tools](https://i.imgur.com/ElEUWak.png)
(`entity_id` will become the ELEMENT_ID parameter)
</details>

## Overview

This repository contains a GitHub Actions workflow and a PowerShell script to generate an ICS calendar file from a WebUntis timetable. The workflow is scheduled to run at specific intervals and can also be triggered manually. The PowerShell script fetches data from a specified URL and generates the ICS file for subscription.

### PowerShell Script: `timeTableToIcs.ps1`

`timeTableToIcs.ps1` generates ICS file(s) from a WebUntis timetable. The script needs headers and cookies that can be exported from the Chrome Dev Console.

#### Parameters

- `baseUrl`: The base URL for the HTTP requests.
- `elementType`: The return type of the request (default is `1`, I don't know what other values might return from the API).
- `elementId`: The ID of the classes timetable. (see [Acquiring Parameters]) 
- `date`: An array of dates (either as strings or DateTime objects) for which to retrieve timetable data. The default is the current week and the next three weeks.
- `OutputFilePath`: The output file path for the ICS file (default is `calendar.ics`).
- `dontCreateMultiDayEvents`: If set, generating "summary" multi-day events will be skipped.
- `overrideSummaries`: A hashtable to override the summaries of the courses. The key is the original course (short)name, the value is the new course name.
- `appendToPreviousICSat`: The path to an existing ICS file to which the new timetable data should be appended.
- `splitByCourse`: If set, the timetable data will be split into separate ICS files for each course.
- `splitByOverrides`: If set, the timetable data will be split into separate ICS files for each course defined in overrideSummaries and the remaining misc. classes.
- `outAllFormats`: If set, the timetable data will be output in all formats.
- `culture`: If set, uses a specific culture for datetime formatting.
- `cookie`: The cookie value for "authentication" (bot-prevention?).
- `tenantId`: The tenant ID for authentication. (see [Acquiring Parameters]) 

### Workflow: `GeneratePage.yml`

The GitHub Actions workflow `GeneratePage.yml` is designed to update and deploy the ICS calendar file to GitHub Pages using the following steps:

1. **Download Previous ICS**: Downloads the previous ICS artifact if available.
2. **Run PowerShell Script**: Executes the PowerShell script to generate a new ICS file.
3. **Compare ICS Files**: Compares the newly generated ICS file with the previous one.
4. **Upload ICS Artifact**: Uploads the newly generated ICS file(s) as an artifact.
5. **Setup Pages**: Configures GitHub Pages for deployment.
6. **Upload Artifact**: Uploads the artifacts as a Pages artifact.
7. **Deploy to GitHub Pages**: Deploys the uploaded artifacts to GitHub Pages.

#### Schedule

The workflow is scheduled to run at the following times:

- Every 6 hours
- Daily at 6:00 UTC
- Daily at 7:00 UTC

..and can also be triggered manually.

### Sample `launch.json`

(for debugging via vscode)

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "PowerShell: Launch Current File",
            "type": "PowerShell",
            "request": "launch",
            "script": "./timeTableToIcs.ps1",
            "args": [
                //"-Verbose",
                //"-Debug",
                //"-ErrorAction", "Inquire",
                "-baseUrl",         "######.webuntis.com",
                "-elementType",     "1",
                "-elementId",       "####", // Course ID, 4 digits long, get from URL via official website
                "-OutputFilePath",  "calendar.ics",
                "-cookie",          "##################", // Get by opening official page with dev tools and copy http request as PowerShell
                "-tenantId",        "#######", // Get by opening official page with dev tools and copy http request as PowerShell
                "-overrideSummaries", "@{\"GK\" = \"GK, Gemeinschaftskunde\";\"LBTL1\" = \"EL, Elektrotechnik\";\"Wi\" = \"Wi, Wirtschafts- und Sozialkunde\";\"E\" = \"EN, Englisch\";\"D\" = \"DE, Deutsch\";\"LBT1\" = \"BWL, Betriebswirtschaftslehre (LBT1/4)\";\"LBT5\" = \"NT, Netzwerktechnik (LBT5)\";\"LBT4\" = \"BWL/ITSY, BWL/ITSY (LBT4/3/1)\";\"LBT2\" = \"SAE, System- und Anwendungsentwicklung (LBT2)\";\"LBT3\" = \"IST, Informations- und Systemtechnik (LBT3)\"}",
                "-appendToPreviousICSat", "./calendar.ics",
                "-outAllFormats",
                "-splitByOverrides"
            ]
        }
    ]
}
```
