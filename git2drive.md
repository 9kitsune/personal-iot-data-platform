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

Below is a highly tactical, pre-built GitHub Actions configuration script. It triggers on a push to any branch, extracts the branch name, cleans it up to prevent filesystem errors, and pushes the file straight to Drive.

```yaml
name: Sync Resume to Google Drive

on:
  push:
    branches:
      - '**'

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Extract Branch Name
        id: extract_branch
        run: |
          BRANCH_NAME=$(echo "${{ github.ref_name }}" | sed 's/\//_/g')
          echo "branch_cleaned=$BRANCH_NAME" >> $GITHUB_OUTPUT

      - name: Upload Markdown to Google Drive
        uses: adityassankarp/google-drive-upload-gitaction@v0.3
        with:
          credentials: ${{ secrets.GDRIVE_CREDENTIALS }}
          filename: 'resume.md'
          folderId: ${{ secrets.GDRIVE_FOLDER_ID }}
          name: 'Resume_${{ steps.extract_branch.outputs.branch_cleaned }}.md'
          overwrite: 'true'
```
What Happens Next?
Once you save this file, commit it, and push it to GitHub, the workflow will trigger immediately.

If you switch branches locally using git checkout -b tailor/mace-data-engineer, modify your resume, and run git push origin tailor/mace-data-engineer, GitHub will fire off this script and you will see a brand new file named Resume_tailor_mace-data-engineer.md appear in your Google Drive folder automatically.
