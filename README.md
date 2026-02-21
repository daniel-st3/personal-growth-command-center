# Personal Growth Command Center (LifeOS)

Hey! This is a personal project I built to help manage my job search, learning goals, and daily habits. I wanted to build something practical that actually solves a problem for me, while also practicing some data engineering logic.

## What is this?
Instead of setting up a full Postgres database and paying for hosting, I decided to use Google Sheets as a lightweight data warehouse. I built a set of automated workflows using n8n (on the free cloud tier) to extract data from my Gmail and Google Calendar, and push it into my Sheets.

This project shows how I handle ETL pipelines, branching logic, and upserts locally without any paid APIs. All the custom data transformations are written in standard JavaScript inside n8n Code nodes.

## Data Structure
I structured my spreadsheet using a basic star schema with fact and dimension tables. Each table is just a separate tab in my Google Sheet:

**Dimensions:**
- `dim_project` (project_id, name, type, priority, status)
- `dim_contact` (contact_id, name, email, company, role, source, stage)
- `dim_habit` (habit_id, name, frequency, target_week)

**Facts:**
- `fact_tasks` (task_id, project_id, created_date, due_date, status, source) -> Depends on `dim_project`
- `fact_applications` (app_id, contact_id, project_id, applied_date, status, role, company) -> Depends on `dim_contact`, `dim_project`
- `fact_habits` (event_id, habit_id, date, status, notes) -> Depends on `dim_habit`
- `fact_learning` (session_id, project_id, skill, date, duration_min, source_url) -> Depends on `dim_project`

**Other:**
- `alerts` (alert_id, date, category, message, status)
- `weekly_summaries` (week_start_date, tasks_done, apps_sent, hours_learning, workouts)

## How to use this project

### 1. Set up the Google Sheet
1. Create a new Google Sheet (I called mine `LifeOS`).
2. Create 9 tabs using the exact names from the Data Structure section above.
3. Add the column names to the first row of each tab and make them bold so n8n recognizes them as headers.
4. Grab your Spreadsheet ID from the URL (the long string of letters and numbers).

### 2. Set up n8n
1. Create a free account on n8n Cloud (or run it locally).
2. Go to Credentials and add your Google permissions (Calendar, Sheets, and Gmail). 

### 3. Import the Workflows
1. In n8n, click "Add Workflow" -> "Import from File" and upload the JSON files from this repo:
   - `GmailJobsToApps.json`: Scans new emails for job application keywords and logs the role/company into my database.
   - `CalendarHabits.json`: Pulls my gym and study sessions from my calendar automatically every hour.
   - `ProjectsSync.json`: Runs every morning to check if I have open tasks across my active projects.
   - `DailyAlertsAndSummary.json`: Sends me an email every morning if I missed my weekly habit goals or need to follow up on a job application.
2. In each workflow, replace the Google Sheets node Document IDs with your own Spreadsheet ID.
3. Activate the workflows and you're good to go!

## Why I built this
I needed a simple way to track my job applications and study progress. By structuring the raw data into fact and dimension tables, I can easily connect the sheet to Looker Studio or Tableau later to build automated dashboards. It was a fun way to apply basic data extraction and pipeline concepts to my actual day-to-day life.
