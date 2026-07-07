# /upskill - Skill Gap Analysis and Learning Plan

You are orchestrating a skill gap analysis for the candidate's job search. The argument (if any) is provided as `$ARGUMENTS`.

This command delegates to the **upskill** skill for all analysis work. Your job here is to parse the input correctly and hand off cleanly.

Follow these steps **exactly in order**.

---

## Step 0: Parse Arguments

Check `$ARGUMENTS`:

- **Empty / no arguments** → **aggregate mode**. Analyse all jobs tracked in `job_search_tracker.csv`.
- **A URL** (starts with `http://` or `https://`) → **targeted mode**. The URL is the job posting to analyse.
- **Anything else** (e.g. a job title, company name, or freeform description) → ask the user to clarify:
  > "Did you mean to pass a job posting URL? If so, paste the full URL. Or run `/upskill` without arguments to analyse all jobs in your tracker."
  Wait for their response before continuing.

Store the mode and URL (if any) — you will reference them throughout.

---

## Step 1: Confirm Scope (aggregate mode only)

In aggregate mode, before starting the analysis, briefly confirm what you will work from:

1. Read `job_search_tracker.csv`.
2. Count the number of rows (excluding the header).
3. Print a one-liner:
   > "Analysing skill gaps across **N tracked jobs**. This may take a moment while I search for study resources."

If the tracker is empty (0 data rows), stop here and tell the user:
> "Your job tracker is empty. Add jobs via `/apply` or by running `/scrape`, then re-run `/upskill`. Alternatively, run `/upskill <URL>` to analyse a specific posting directly."

In **targeted mode**, skip this step and go straight to Step 2.

---

## Step 2: Run the Upskill Skill

Invoke the **upskill** skill. The skill's SKILL.md (`.claude/skills/upskill/SKILL.md`) contains the full analysis workflow — follow it from **Step 1** onwards, passing the mode and URL you determined in Step 0.

The skill handles:
- Loading profile and tracker data
- Hard skill diff (Pass 1) and LLM synthesis (Pass 2)
- Building and printing the gap heatmap
- Searching the web for study resources
- Writing the final report to `upskill/`

Do not duplicate the skill's steps here. Invoke it and let it run to completion.

---

## Step 3: Present Next Steps

After the skill saves its report, present a short summary to the user:

> **`/upskill` complete.**
>
> - Report saved to `upskill/<filename>.md`
> - **Critical gaps:** [list gap names, or "none"]
> - **High gaps:** [list gap names, or "none"]
> - **Total estimated study time:** ~Xh
>
> **What to do next:**
> - Open the report to review the full learning plan and suggested study order.
> - Run `/upskill <URL>` to drill into a specific posting's gaps.
> - Run `/scrape` or `/apply` to continue your job search.

If the report is in targeted mode, replace the job count line with the job title and company:
> **Target:** `<Role>` at `<Company>`

---

## Design Notes

- This command is a thin orchestration layer. All analysis logic lives in `.claude/skills/upskill/SKILL.md` — keep it there, not here.
- Aggregate mode is the default and the most useful for ongoing career planning.
- Targeted mode is useful for a single speculative application where you want to know your gaps before committing to `/apply`.
- The command is idempotent: re-running it in aggregate mode produces a new dated report and computes a diff against the previous one (handled by the skill's Step 8).
