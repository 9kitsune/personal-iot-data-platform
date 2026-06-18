# The GitHub-to-Drive Automation Pipeline (CI/CD Approach)

This approach treats GitHub as the source of truth for version control and uses automated pipelines to push copies or rendered versions of the resume into Google Drive.

Tactical Execution:

1. Maintain your Markdown resume in a private or public GitHub repository.

2. Use standard Git workflows (main branch for the master copy, feature branches like tailor/mace-analytics for variations).

3. Set up a GitHub Actions workflow that triggers on a push to any branch.

4. Configure the workflow to use a step like google-github-actions/upload-cloud-storage or a customized script using Google Drive API credentials to upload the .md file (or a compiled PDF) to a specific Google Drive folder.

5. Program the workflow to name the file based on the branch name (e.g., Resume_main.md or Resume_tailor_mace-analytics.md).

Pros: This cleanly isolates your version control metadata from Google Drive's synchronization engine and provides automated backups on both platforms.

# Workflow v.1

Let's set up the GitHub Actions pipeline. This approach keeps the version control completely standard and clean on GitHub, while automatically delivering a tailored, neatly named Markdown file (or rendered PDF) directly into the Google Drive folder whenever push a branch.

Here is the tactical blueprint to get this pipeline built and authenticated.

## The Workflow Blueprint
To make this work safely, we need a GitHub Actions workflow that authenticates with Google Drive using a Service Account (this avoids messy interactive OAuth logins) and uploads the file using the GitHub branch name as a prefix.

```
[ Your Local Git ] 
         │
         ▼ (git push branch/mace-data-eng)
[ GitHub Repository ]
         │
         ▼ (Triggers GitHub Action)
[ Google Drive API ] ──► Uploads to: "Resume_branch_abc-data-eng.md"
``` 
# Step-by-Step Execution Plan

Step 1: Set Up the Google Cloud Service Account
We need a machine-to-machine key so GitHub can talk directly to your Google Drive.
* 1.Create a Google Cloud Project:
  * Requires a standard Google account.
  * Go to the Google Cloud Console. Create a new project named Resume-Sync-Automation.
* 2.Enable the Google Drive API:
  * Enables programmatic access.
  * In the search bar at the top, type Google Drive API, click it, and hit Enable.
* 3.Create Service Account Credentials:
  * Generates the secret key.
  * Navigate to IAM & Admin > Service Accounts. Click Create Service Account. Give it a name like github-resume-uploader. Click Create and Continue, skip the optional roles, and hit Done.
* 4.Download the JSON Key:
  * Keep this private.
  * Click on your newly created service account, go to the Keys tab, click Add Key > Create new key, select JSON, and download the file.

Step 2: Prepare Your Google Drive Target Folder
Because a Service Account is essentially an isolated "robot user," it cannot see your personal Google Drive unless you explicitly give it access.
* 1.Open your Google Drive in a browser and create a dedicated folder (e.g., Resume Branches).
* 2.Copy the Folder ID from the URL bar. It is the long string of letters and numbers right after /folders/ (for example, 1A2b3C4d...). Save this for later.
* 3.Open the downloaded JSON key file, find the "client_email" field (it will look like github-resume-uploader@...iam.gserviceaccount.com), and copy it.
* 4.Go back to your Google Drive folder, click Share, paste that client email, and grant it Editor permissions.

Step 3: Configure GitHub Secrets
Never hardcode your Google credentials into your public or private repository files. We will store them securely in GitHub's backend.
* 1.Go to your GitHub repository where the MD resume lives.
* 2.Navigate to Settings > Secrets and variables > Actions.
* 3.Click New repository secret and add these two secrets:
  * GDRIVE_CREDENTIALS: Open your downloaded JSON key file, copy the entire raw text content, and paste it here.
  * GDRIVE_FOLDER_ID: Paste the long string Folder ID you copied from your Google Drive URL.

Step 4: Add the GitHub Actions Workflow File
In the root directory of your local resume repository, create a file named exactly .github/workflows/sync_to_drive.yml.

Below is a highly tactical, pre-built GitHub Actions configuration script. It triggers on a push to any branch, extracts the branch name, cleans it up to prevent filesystem errors, and pushes the file straight to Drive. It also doesn't use any 3rd party marketplace plugin, so it is dependency-free, and secure-by-design.

```yaml
name: Sync Resume to Google Drive (Zero Dependencies)

on:
  push:
    branches:
      - '**' # Triggers on push to main or any feature branch

jobs:
  build-and-upload:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Extract Branch Name
        id: extract_branch
        run: |
          BRANCH_NAME=$(echo "${{ github.ref_name }}" | sed 's/\//_/g')
          echo "branch_cleaned=$BRANCH_NAME" >> $GITHUB_OUTPUT

      - name: Install Compilers and Python Auth Drivers
        run: |
          sudo apt-get update
          sudo apt-get install -y pandoc weasyprint python3-pip python3-setuptools
          # Install the official Google Auth library directly so we can securely generate the token
          pip3 install google-auth requests

      - name: Compile Markdown to Styled PDF
        run: |
          cat << 'EOF' > style.css
          @page { size: A4; margin: 20mm 15mm; }
          body { font-family: 'Helvetica Neue', Arial, sans-serif; font-size: 10.5pt; line-height: 1.5; color: #2D3748; }
          h1 { font-size: 20pt; color: #1A365D; text-transform: uppercase; margin-bottom: 5px; }
          h3 { font-size: 13pt; color: #2B6CB0; border-bottom: 1px solid #E2E8F0; padding-bottom: 3px; margin-top: 20px; }
          p, li { margin-bottom: 6px; text-align: justify; }
          ul { padding-left: 20px; }
          hr { border: 0; border-top: 1px solid #CBD5E0; margin: 15px 0; }
          .job-entry { page-break-inside: avoid; }
          EOF

          pandoc resume.md -s -c style.css -o resume.html
          weasyprint resume.html resume.pdf

      - name: Generate OAuth Access Token & Upload via Curl
        env:
          GDRIVE_CREDENTIALS: ${{ secrets.GDRIVE_CREDENTIALS }}
          FOLDER_ID: ${{ secrets.GDRIVE_FOLDER_ID }}
          BRANCH_NAME: ${{ steps.extract_branch.outputs.branch_cleaned }}
        run: |
          # 1. Write the credentials JSON out to a temporary local file
          echo "$GDRIVE_CREDENTIALS" > sa_key.json
          
          # 2. Use a short inline Python script to generate a secure Google OAuth access token
          ACCESS_TOKEN=$(python3 -c "
          import json
          from google.oauth2 import service_account
          import google.auth.transport.requests
          
          scopes = ['https://www.googleapis.com/auth/drive.file']
          creds = service_account.Credentials.from_service_account_file('sa_key.json', scopes=scopes)
          request = google.auth.transport.requests.Request()
          creds.refresh(request)
          print(creds.token)
          ")
          
          # Clean up the secret key from the disk space immediately
          rm sa_key.json
          
          # 3. Define target filenames
          MD_NAME="Resume_${BRANCH_NAME}.md"
          PDF_NAME="Resume_${BRANCH_NAME}.pdf"
          
          # 4. Upload Markdown file via an explicit Google Drive API Multipart Curl request
          curl -X POST -L \
            -H "Authorization: Bearer $ACCESS_TOKEN" \
            -F "metadata={name: '$MD_NAME', parents: ['$FOLDER_ID']};type=application/json;charset=UTF-8" \
            -F "file=@resume.md;type=text/markdown" \
            "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart"
            
          # 5. Upload PDF file via an explicit Google Drive API Multipart Curl request
          curl -X POST -L \
            -H "Authorization: Bearer $ACCESS_TOKEN" \
            -F "metadata={name: '$PDF_NAME', parents: ['$FOLDER_ID']};type=application/json;charset=UTF-8" \
            -F "file=@resume.pdf;type=application/pdf" \
            "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart"
```
## The Architecture: "Feature" Branches for Archetypes
Instead of treating branches as individual applications, treat them as skill-set archetypes. Keep long-lived branches tracking the distinct professional hats you wear:
* main: The definitive master copy. Every piece of raw experience, every credential, and every award goes here. It is your single source of truth, completely un-truncated.
* role/data-engineer: Tailored heavily toward Python, PySpark, orchestration (dbt/Airflow), performance tuning, and infrastructure.
* role/analytics-engineer: Tailored toward data integrity, data warehouse modeling, semantic modeling layers, and bridging architecture with direct business logic.
* role/data-architect: Focused entirely on high-level enterprise design patterns, governance, platforms (like Fabric), standards-setting, and strategic advisory.

## The Workflow: How to Handle Specific Companies
* 1.find a specific role at a company, just use the workflow tools to execute an elegant checkout pattern:
```
┌──► role/data-engineer ─────────► (Builds generic DE PDF)
                  │
[ main ] ─────────┼──► role/analytics-engineer ────► (Builds generic AE PDF)
(Core Data Estate)│
                  └──► role/data-architect ────────► (Apply subtle copy tweaks)
                                │
                                └───► git commit -m "Company ABC: Emphasize platform leadership"
```
* 1.Identify the core requirement: Decide if the target job is closer to a Data Engineer, Analytics Engineer, or Architect role.
* 2.Check out that specific role branch: git checkout role/data-architect
* 3.Make minor, surgical changes: Read the job description, spot their specific keywords, and tweak a bullet point or two directly in the file.
* 4.Commit with context: Commit your changes with a clear descriptive message:
* ```
  git commit -am "Tailor summary keywords for Mace Group submission"
  ```
* 5.Let the Pipeline Run: GitHub Actions automatically pushes your updated Markdown and styled PDF files to Google Drive under the file name Resume_role_data-architect.pdf.

Pros: 
* **Easy Upstream Merges**: When you pass a exam, earn a new certification, or finish a massive platform modernization project at work, you add it once to your main branch. You can then cleanly merge main down into your role branches (role/data-engineer, etc.) without managing a tangled web of 40 dead company branches.
* **Continuous Improvement**: Every time you refine the phrasing on your role/data-architect branch to make it punchier for one application, that value automatically upgrades your baseline template for the next time you apply for an architect role.
