# Gradebook Merge

Matches students from a LMS roster to their grades on Gradescope, calculates their final grade alongside grading policies, and then outputs a CSV file of students and their updated grades.

## Table Of Contents
- [Features](#features)
- [Install](#install)
- [Usage](#usage)
  - [CLI](#cli)
     - [Help](#help)
     - [Required Arguments](#required-arguments)
     - [Important Argument](#important-argument)
     - [Common Error](#common-error)
     - [Check Inputs](#check-inputs)
     - [Match Report](#match-report)
     - [Configuration Setup](#configuration-setup)
     - [Merge](#merge)
- [API](#api)
  - [Python Library](#python-library)
  - [Custom Policy](#custom-policy)
  - [References](#references)
    - [Loading Functions](#loading-functions)
    - [Matching Functions](#matching-functions)
    - [Policy Functions](#policy-functions)
    - [Create Functions](#create-functions)
    - [Utility Functions](#utility-functions)
    - [Data Types](#data-types)
    - [Errors](#errors)
- [Maintainers](#maintainers)
- [Credits](#credits)
---

## Features
- **Automatic Student Matching** – Matches students to their Gradescope submissions using student ids, emails, and names.
- **Customizable Grading Policy** – Calculates grades against configurable rules such as late penalties, dropping lowest grades, and rounding grades.
- **Dry Run** – Allows a preview of the grades if updated without commiting them or outputing a CSV file.

---

## Install

Install Python (3.10 or newer): https://www.python.org/downloads/
- Verify (In Command Line): python --version

Install Git: https://git-scm.com/install/windows
- Verify (In Command Line): git --version

Open Repository in Command Line:
```bash
# Clone the repository
git clone https://github.com/KeeganGallagher5959/CICS-Writing-Project-README.git

# Navigate to the project
cd CICS-Writing-Project-README

# Install dependencies
python -m pip install .
```
- After installing the following warning may pop up looking similar to this:
```
WARNING: The script gbmerge.exe is installed in 'C:{YOUR PATH}' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
```
- Copy your version of 'C:{YOUR PATH}' inside single quotation (will be different based on where you cloned repository, python version, etc).
```bash
# Set Path
set "PATH=%PATH%;C:{YOUR PATH}"
```
- Verify (In Command Line): gbmerge --version
- If you get a gbmerge version then the installation is complete. 
---

## Usage 
To use this application you need two files, the LMS gradebook CSV file and the Gradescope CSV file. Make sure to have these two files downloaded and in your `Downloads` folder.

In the repository directory, after having downloaded both your CSV files, perform the following commands:
```bash
# Copy your roster into repository
copy "%USERPROFILE%\Downloads\{ROSTER FILE NAME}.csv" .

# Create an export folder for your grade scores
mkdir exports

# Copy your grade scores into exports folder
copy "%USERPROFILE%\Downloads\{GRADE SCORE FILE NAME}.csv" exports\
```

### CLI

In the following examples our `{ROSTER FILE NAME}.csv` will be `roster.csv` and our `{GRADE SCORE FILE NAME}.csv` will be `hw4_scores.csv`.

#### Help

If the user wants to see the four sub commands or examples of commands that are commonly used in this process they can use the following command to do so.

**Command:**
```bash
gbmerge --help
```

**Output:**
```text
usage: gbmerge [-h] [--version] COMMAND ...

Merge Gradescope exports into an LMS gradebook upload CSV.

positional arguments:
  COMMAND
    merge         write the upload CSV
    check         validate the inputs without writing anything
    match-report  show how students matched, and who did not
    init-config   write a starter config file

options:
  -h, --help      show this help message and exit
  --version       show program's version number and exit

examples:
  gbmerge check --roster roster.csv --assignment "Homework 4 (884213)"
  gbmerge merge --roster roster.csv --assignment "Homework 4 (884213)" --dry-run
  gbmerge merge --roster roster.csv --assignment "Homework 4 (884213)" \
      --exports exports/hw4_scores.csv --out upload.csv --audit-log audit.jsonl

the assignment name is the LMS column header, quoted, including the id in
brackets. it is not the name of the gradescope assignment.
```

#### Required Arguments

If a user attempts to run a command without the required arguments the user will receive a warning about the missing requirments.

**Command:**
```bash
gbmerge merge --roster roster.csv  --dry-run 
```

**Output:**
```text
usage: gbmerge merge [-h] [--config PATH] [--encoding NAME] [--json] [-v] --roster PATH [--exports PATH] --assignment NAME [--match-on LIST] [--fuzzy]
                     [--on-ambiguous {report,skip,first,fail}] [--out PATH] [-n] [--policy NAME] [--late-penalty FRACTION] [--max-late-days N]
                     [--drop-lowest N] [--round {none,int,half_up,half_even,tenth}] [--exempt-id ID] [--exempt-file PATH] [--on-missing {skip,zero,fail}]
                     [--audit-log PATH] [-y]
gbmerge merge: error: the following arguments are required: --assignment
```
This tells us we are missing the required assignment argument.

#### Important Argument
If a single file is in the user's export folder then commands will default to that one without the export argument.

If there are multiple files inside the exports folder it is important to distinguish which one the user is targeting, the following command is an example of making sure the command targets the grades from the `hw4_scores.csv` in the exports folder.

**Command:**
```bash
gbmerge merge --roster roster.csv --assignment "Homework 4 (884213)" --exports exports/hw4_scores.csv --dry-run
```

#### Common Error

Remember to have the homework assignment in double quotations. 

**Command:**
```bash
gbmerge merge --roster roster.csv --assignment Homework 4 (884213) --dry-run
```

**Output:**
```text
Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "C:\Users\keega\AppData\Roaming\Python\Python313\Scripts\gbmerge.exe\__main__.py", line 7, in <module>
    sys.exit(main())
             ~~~~^^
  File "C:\Users\keega\AppData\Roaming\Python\Python313\site-packages\gbmerge\cli.py", line 728, in main
    return handler(args)
  File "C:\Users\keega\AppData\Roaming\Python\Python313\site-packages\gbmerge\cli.py", line 444, in cmd_merge
    plan = job.plan()
  File "C:\Users\keega\AppData\Roaming\Python\Python313\site-packages\gbmerge\writer.py", line 558, in plan
    return build_upload_plan(
        gb.roster,
    ...<3 lines>...
        keep_test_student=bool(self._setting("keep_test_student")),
    )
  File "C:\Users\keega\AppData\Roaming\Python\Python313\site-packages\gbmerge\writer.py", line 237, in build_upload_plan
    before = _current_values(roster, assignment_column)
  File "C:\Users\keega\AppData\Roaming\Python\Python313\site-packages\gbmerge\writer.py", line 210, in _current_values
    return [row.data[assignment_column] for row in roster.rows]
            ~~~~~~~~^^^^^^^^^^^^^^^^^^^
KeyError: 'Homework'
```
Having quotations around the homework assignment will have it run as normal.

**Command:**
```bash
gbmerge merge --roster roster.csv --assignment "Homework 4 (884213)" --dry-run
```

**Output:**
```text
assignment column: 'Homework 4 (884213)'
rows in file:      5 (3 students + 2 kept non-student rows)
cells changed:     3 (2 new, 1 overwriting an existing grade)
skipped:           0 students with no submission
late:              0 (0 penalized, 0 exempt)
scored zero:       0
policy:            policy=standard, 10%/started day, max 3 days then zero, round=half_up

changes:
  Alice Smith (31402118): (blank) -> 95
  Bob Jones (31402119): (blank) -> 88
  Charlie Brown (31402120): 75 -> 92

note: 1 of these replace a grade the LMS already has. There is no undo.

dry run: nothing written.
```

#### Check Inputs

It is important before starting any merges to check that there is no issues with your CSV files, if no problems are found we are good to perform a dry run. If there are issues on the checking stage that means there are issues with the file data itself.

**Command:**
```bash
gbmerge check --roster roster.csv --exports exports/hw4_scores.csv --assignment "Homework 4 (884213)"
```

**Output:**
```text
ok    roster roster.csv: 3 students, 2 non-student rows kept, decoded as utf-8-sig
ok    export exports\hw4_scores.csv: 3 submissions, 0 questions, decoded as utf-8-sig
ok    assignment column 'Homework 4 (884213)': found
ok    matching: 3 of 3 students matched from 3 submissions
ok    policy: policy=standard, 10%/started day, max 3 days then zero, round=half_up
ok    no config file in use; built-in defaults apply. 'gbmerge init-config' writes one

no problems found
```

#### Match Report

The check command only looks to see if the files are valid, not how the data within them will work together. The Match Report command allows the user to see if any students remain unmatched and if so they can see the reason why with the use of the explain argument.

**Command:**
```bash
gbmerge match-report --roster roster.csv --exports exports/hw4_scores.csv --match-on sis_id,email --explain
```

**Output:**
```text
matched 3 of 3 students from 3 submissions (strategies: sis_id, email)
  by strategy: sis_id: 3
```
The above command used the match-on attribute, this allows the user to specify by what attribute they want the students to be matched on and by which order of priority. For instance in this example match-on tries to match student by id first and email second, if the argument was instead `--match-on email,sis_id` then it'd do the opposite and match by email first. 

The output shows that all students were successfully matched by id.

#### Configuration Setup

Grading rules can be applied to the grading calculations during merge through creating a configuration file. If no configuration files exists a default configuration to calculate grades are used. The following command is how a user can create there own configuration.

**Command:**
```bash
gbmerge init-config
```

**Output:**
```text
wrote gbmerge.yml
every key in it is a default; delete the ones you do not need.
```
To change the configuration the user must edit the file manually. Doing this command again will state that one already exists, add the attribute `--yes` to the end of the command to overwrite the configuration file (This is a fast way to reset all of the configuration rules back to default).

#### Merge

Before actually merging make sure to perform a dry run on what you want to merge with the following command.

**Command:**
```bash
gbmerge merge --roster roster.csv --exports exports/hw4_scores.csv --assignment "Homework 4 (884213)" --dry-run
```

**Output:**
```text
assignment column: 'Homework 4 (884213)'
rows in file:      5 (3 students + 2 kept non-student rows)
cells changed:     3 (2 new, 1 overwriting an existing grade)
skipped:           0 students with no submission
late:              0 (0 penalized, 0 exempt)
scored zero:       0
policy:            policy=standard, 10%/started day, max 3 days then zero, round=half_up

changes:
  Alice Smith (31402118): (blank) -> 95
  Bob Jones (31402119): (blank) -> 88
  Charlie Brown (31402120): 75 -> 92

note: 1 of these replace a grade the LMS already has. There is no undo.

dry run: nothing written.
```
As stated in the output nothing is actually merged and, importantly, that after an actual merge there is no undoing. If a grade is changed due to a merge by mistake undoing it isn't a functionality of this application.

Either way, when the changes in the dry run look correct perform the following command to do the actual merge.

**Command:**
```bash
gbmerge merge --roster roster.csv --exports exports/hw4_scores.csv --assignment "Homework 4 (884213)" --match-on sis_id,email --out upload.csv --audit-log audit.jsonl
```

**Output (1/2):**
```text
assignment column: 'Homework 4 (884213)'
rows in file:      5 (3 students + 2 kept non-student rows)
cells changed:     3 (2 new, 1 overwriting an existing grade)
skipped:           0 students with no submission
late:              0 (0 penalized, 0 exempt)
scored zero:       0
policy:            policy=standard, 10%/started day, max 3 days then zero, round=half_up

changes:
  Alice Smith (31402118): (blank) -> 95
  Bob Jones (31402119): (blank) -> 88
  Charlie Brown (31402120): 75 -> 92

note: 1 of these replace a grade the LMS already has. There is no undo.
Write 3 changed grades to upload.csv, replacing 1 existing grades? [y/N]
```
Input y to continue or N to abort then hit enter.

**Output (2/2):**
```text
wrote upload.csv (3 changed, 5 rows)
audit log appended to audit.jsonl
```

The `--out` attribute designates what the user wants the outputted CSV file created form the merge to be called. This file contains the updated grade data that can be uploaded to the grade book. It should not be edited past this point.

The `--audit-log` attribute designates what the user wants the outputted json1 file created from the merge to be called. This file contains information about the merge that can be looked back on in review such as grade changes, warnings, etc.

---


## API

### Python Library

Instead of using Gradebook Merge through command lines you can alternatively interact with it in Python with the following code example:

```python
from gbmerge import MergeJob, load_config

job = MergeJob(
    roster="roster.csv",
    exports="exports/hw4_scores.csv",
    assignment="Homework 4 (884213)",
    config=load_config()
)

plan = job.plan()
print(plan.format_preview())
job.write(plan, "upload.csv")
```

- `MergeJob` manages loading, matching, scoring, and generating output.
- `load_config()` loads configuration settings or built-in defaults.
- `plan()` returns an `UploadPlan` without writing a file.
- `format_preview()` displays the proposed grade changes.
- `write()` saves the upload CSV and returns its path.

To perform a dry run in Python change the last three lines to these two:

```python
plan = job.run(dry_run=True)
print(plan.format_preview())
```

### Custom Policy

The `register_policy()` function lets users make their own grading rules.

Example, the following applies the policy to give late submissions half credit while giving on-time and exempt students full credit:

```python
from gbmerge import MergeJob, load_config, register_policy

@register_policy("half_credit_late")
def half_credit_late(ctx):
    if ctx.exempt or not ctx.is_late:
        return ctx.raw_score
    return ctx.raw_score * 0.5

config = load_config()
config["policy"] = "half_credit_late"

job = MergeJob(
    roster="roster.csv",
    exports="exports/hw4_scores.csv",
    assignment="Homework 4 (884213)",
    config=config
)

plan = job.run(dry_run=True)
print(plan.format_preview())
```

The policy function (Ex: `half_credit_late(ctx)` in the above code) receives a `ScoreContext` object containing information about the student's submission:

- `ctx.raw_score` — Original assignment score.
- `ctx.is_late` — Whether the submission was late.
- `ctx.exempt` — Whether the student is exempt from late penalties.

### References

#### Loading Functions

| Name | Description / Output Value |
|---|---|
| `load_roster(path, ...)` | Loads a LMS gradebook CSV and outputs a `Roster`. |
| `load_export(path, ...)` | Loads a Gradescope CSV and outputs a `GradescopeExport`. |
| `load_exports(directory, ...)` | Loads matching Gradescope CSVs and outputs a list of `GradescopeExport` objects. |
| `load_config(path=None, ...)` | Loads configuration and outputs a `Config`. |

#### Matching Functions

| Name | Description / Output Value |
|---|---|
| `match_students(roster, exports, ...)` | Matches submissions to students and outputs a `MatchReport`. |
| `parse_match_on(value)` | Validates matching strategies and outputs a list of strategy names. |
| `explain_unmatched(report, roster)` | Outputs a list of explanations for unmatched submissions. |

#### Policy Functions

| Name | Description / Output Value |
|---|---|
| `apply_policy(roster, report, policy=None, ...)` | Applies grading rules and outputs a `ScoreSheet`. |
| `register_policy(name, fn=None)` | Registers a custom policy function. |
| `get_policy(name)` | Outputs a registered policy function. |
| `available_policies()` | Outputs a sorted list of available policy names. |
| `late_days(lateness_seconds, grace_minutes=0)` | Outputs the number of started late days. |
| `apply_rounding(value, mode="half_up", places=2)` | Outputs a rounded numeric score. |
| `drop_lowest_questions(question_scores, count)` | Outputs the remaining score total and a list of dropped questions. |

#### Create Functions

| Name | Description / Output Value |
|---|---|
| `build_upload_plan(roster, sheet, assignment_column, ...)` | Builds an in-memory `UploadPlan`. |
| `write_upload_csv(path, plan, ...)` | Writes an LMS upload CSV and outputs its path. |
| `write_audit_log(path, plan, ...)` | Writes a JSON-lines audit log and outputs its path. |

####  Utility Functions

| Name | Description / Output Value |
|---|---|
| `normalize_email(value)` | Outputs the lowercase email prefix |
| `normalize_sis_id(value)` | Outputs a normalized SIS ID |
| `detect_excel_damage(fieldnames, rows, ...)` | Outputs warnings about possible CSV damage |
| `find_config(start=".")` | Searches for a configuration file and outputs its path or `None`. |

#### Data Types

| Name | Description |
|---|---|
| `MergeJob` | Represents a merge operation. Provides `load()`, `match()`, `score()`, `plan()`, `write()`, and `run()` methods. |
| `Gradebook` | Represents a loaded gradebook. Provides `load()`, `match()`, `score()`, and `plan()` methods. |
| `Roster`, `RosterRow` | Gradebook and student information |
| `GradescopeExport`, `Submission` | Gradescope export and submission information |
| `Config`, `DEFAULT_CONFIG` | Configuration settings |
| `MatchReport`, `Match`, `Ambiguity`, `Duplicate` | Student matching results |
| `Policy`, `PolicyOutcome`, `ScoreContext` | Grading policy information |
| `ScoreSheet`, `ScoredStudent` | Calculated grades |
| `UploadPlan`, `Change` | Output CSV and proposed changes |


#### Errors

| Exception | Description |
|---|---|
| `ConfigError` | Invalid configuration |
| `RosterError` | Invalid LMS gradebook |
| `ExportError` | Invalid Gradescope export |
| `EncodingError` | File encoding failure |
| `ColumnNotFoundError` | Required column is missing |
| `MatchError` | Invalid matching strategy |
| `AmbiguousMatchError` | Multiple possible student matches |
| `PolicyError` | Invalid grading policy |
| `WriteError` | Output file could not be written |
| `KeyError` | Invalid column name |

---
## Maintainers

Keegan Gallagher - ksgallagher@umass.edu

---

## Credits

This README was assisted in its creation with the use of chatgpt to help with markdown formatting and the making test files.
