# Test Strategy
## Objective
Verify that creating a localised role profile branch and pushing a change successfully triggers the native GitHub Actions runner to generate a beautifully styled PDF and clean Markdown file, uploading them exclusively to the target Google Drive directory without authentication failures or visual formatting degradation.

## Scope of Validation
* 1.Git Isolation: Ensure changes made to the "Senior Data Engineer" profile do not affect the main branch.
* 2.Compilation Integrity: Ensure Pandoc and WeasyPrint successfully parse the Markdown updates (including the new initiative description) and render an A4 PDF without breaking margins.
* 3.API & Pipeline Security: Confirm the Python inline authenticator successfully generates a short-lived token and that curl transmits the payloads natively.
* 4.Target Directory Delivery: Confirm both artifacts appear in the hidden target folder with correct naming syntax (Resume_role_senior-data-engineer.pdf).

## Test Execution Steps
### Phase 1: Local Git Setup & Branching
Run these commands in your local resume repository terminal.
```
# 1. Ensure you are on main and completely up to date
git checkout main
git pull origin main

# 2. Create and switch to your new long-lived role branch
git checkout -b role/senior-data-engineer
```
### Phase 2: Make the Tactical Resume Edits
Open your resume.md file in your text editor. Locate your professional summary or your current role block (e.g., your tenure at Skanska UK) and apply your targeted updates.

Example Content Update to Paste In:
```
### Senior Analytics Engineer | Company ABC UK (June 2022 - Present)
* Leading a strategic data modernisation initiative to migrate legacy, siloed Microsoft Fabric notebooks into a centralised, dbt-centric transformation pipeline, standardising testing and documentation across the engineering lifecycle.
* Actively completing final preparation for the Microsoft DP-600 (Fabric Analytics Engineer Associate) certification (Exam window pending) to formalise expert-level mastery over enterprise semantic modeling and OneLake architectures.
```
Save and close the resume.md file.

### Phase 3: Commit and Trigger the Pipeline
Stage your changes and push them upstream. This push is what instructs GitHub to fire up the zero-dependency script.
```
# 1. Stage the modified resume
git add resume.md

# 2. Commit with clean context
git commit -m "feat: emphasize dbt migration framework and pending DP-600 status"

# 3. Push the branch to GitHub for the first time
git push -u origin role/senior-data-engineer
```
### Phase 4: Monitor the Live Automation Run
* 1.Open your browser and navigate to your GitHub repository page.
* 2.Click on the Actions tab at the top.
* 3.You will see a live workflow run spinning up titled: Sync Resume to Google Drive (Zero Dependencies). Click into it.
* 4.Expand the log logs for the Generate OAuth Access Token & Upload via Curl step.
* 5.Verify that it prints no authorization errors, prints standard HTTP upload codes (like 200 OK), and safely clears the sa_key.json file from disk.

### Phase 5: Google Drive Verification (The Final Gate)
Open your private Google Drive target directory (the one mapped to your FOLDER_ID). Check for the following success states:

* [ ] Two new files must exist: Resume_role_senior-data-engineer.md and Resume_role_senior-data-engineer.pdf.

* [ ] Open the PDF file directly inside Google Drive. Scroll down to your current experience block and verify that your new bullet points regarding the dbt-centric pipeline and pending DP-600 exam render cleanly without clipping text boundaries or creating awkward orphan lines.

* [ ] Jump back to your local terminal, run git checkout main, and open resume.md. Verify that your master resume remains completely untouched by these edits.

Once those checkmarks pass, your version-controlled data estate is fully operational.
