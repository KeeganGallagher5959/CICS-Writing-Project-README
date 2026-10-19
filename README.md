# Gradebook Merge

Matches students from a LMS roster to their grades on Gradescope, calculates their final grade alongside grading policies, and then outputs a CSV file of students and their updated grades.

## Table Of Contents
- [Features](#features)
- [Install](#install)
- [Usage](#usage)
  - [CLI](#cli)
- [API](#api)
- [Maintainers](#maintainers)
- [Credits](#credits)
---

## Features
- **Automatic Student Matching** – Matches students to their Gradescope submissions using student ids, emails, and names.
- **Customizable Grading Policy** – Calculates grades against configurable rules such as late penalties, dropping lowest grades, and rounding grades.
- **Dry Run** – Allows a preview of the grades if updated without commiting them or outputing a CSV file.

---

## Install

Install Python (3.10 or newer): https://www.python.org/downloads/ \
Verify (In Command Line): python --version

Install Git: https://git-scm.com/install/windows \
Verify (In Command Line): git --version

Open Repository in Command Line:
```bash
# Clone the repository
git clone https://github.com/KeeganGallagher5959/CICS-Writing-Project-README.git

# Navigate to the project
cd CICS-Writing-Project-README

# Install dependencies
python -m pip install .
```
Verify (In Command Line): gbmerge --version

---

## Usage
<!-- TODO -->

```

### CLI
<!-- TODO -->
```

---


## API

### Python Library

Instead of using the API through command lines you can alternatively use it in Python with the following code:

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
| `load_roster(path, ...)` | Loads an LMS gradebook CSV and outputs a `Roster`. |
| `load_export(path, ...)` | Loads one Gradescope CSV and outputs a `GradescopeExport`. |
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
| `Roster`, `RosterRow` | Gradebook and student information |
| `GradescopeExport`, `Submission` | Gradescope export and submission information |
| `Config`, `DEFAULT_CONFIG` | Configuration settings |
| `MatchReport`, `Match`, `Ambiguity`, `Duplicate` | Student matching results |
| `Policy`, `PolicyOutcome`, `ScoreContext` | Grading policy information |
| `ScoreSheet`, `ScoredStudent` | Calculated grades |
| `UploadPlan`, `Change` | Output CSV and proposed changes |

### Errors

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

The creation of this README used chatgpt to help with markdown formatting.
