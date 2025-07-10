# Keep Your MEGA Accounts Alive with GitHub Actions 
[AI GENERATED]

This guide explains how to use the `keep-mega-alive` GitHub repository to automatically log in to your MEGA accounts, preventing them from being deactivated due to inactivity. We will cover two methods:

1.  **Basic Setup:** Using the built-in GitHub Actions scheduler. This is simple but may be disabled by GitHub on inactive repositories after 60-90 days.
2.  **Advanced Setup:** Using a free Cloudflare Worker to trigger the GitHub Action. This ensures the script runs indefinitely, even on an inactive repository.

## Method 1: Basic Setup with GitHub Actions

This method is the quickest way to get started.

### Step 1: Fork the Repository

First, you need to create your own copy of the repository.

1.  Go to the original repository: `https://github.com/MS-Jahan/keep-mega-alive`
2.  In the top-right corner of the page, click the **Fork** button.
3.  This will create a copy of the repository under your own GitHub account (e.g., `your-username/keep-mega-alive`).

### Step 2: Add Your MEGA Credentials as a Secret

To allow the script to log in, you must securely provide your account credentials. We will use GitHub's encrypted secrets for this.

1.  In your newly forked repository, go to the **Settings** tab.
2.  In the left sidebar, navigate to **Secrets and variables** > **Actions**.
3.  Click the **New repository secret** button.
4.  For the **Name**, enter exactly:
    ```
    MEGA_PASSWORDS_CSV
    ```
5.  In the **Secret** box, enter your MEGA account emails and passwords. Each account should be on a new line, with the email and password separated by a comma.

    **Format:** `email,password`

    **Example:**
    ```
    my.first.account@email.com,MySecurePassword1
    another-account@yahoo.com,AnotherPassw0rd!
    mega_user@proton.me,P@ssw0rd321
    ```
6.  Click **Add secret**.

### Step 3: (Optional) Add a Healthchecks.io URL

You can use a free service like [Healthchecks.io](https://healthchecks.io) to get notified whenever your script runs successfully.

1.  Sign up or log in to [Healthchecks.io](https://healthchecks.io) and create a new check.
2.  Copy the **Ping URL** provided for your new check.
3.  Go back to your GitHub repository's secrets page (`Settings` > `Secrets and variables` > `Actions`).
4.  Create another **New repository secret**.
5.  For the **Name**, enter:
    ```
    PING_URL
    ```
6.  Paste the URL you copied from Healthchecks.io into the **Secret** box and click **Add secret**.

> **Note:** If you skip this step, the script will still work, but the "Ping Healthchecks" part of the workflow will fail. This is harmless.

### Step 4: Run and Verify the Workflow

The workflow is scheduled to run automatically every Sunday. However, you should run it manually the first time to ensure everything is configured correctly.

1.  Go to the **Actions** tab in your repository.
2.  In the left sidebar, click on the **Keep Mega Alive** workflow.
3.  You will see a message: "This workflow has a `workflow_dispatch` event trigger." Click the **Run workflow** button on the right.
4.  A new workflow run will start. Click on it to see the progress.
5.  Once the job completes, you can inspect the logs. Expand the **Run script** step to see the login process for each account. A successful run will show the used storage and log out messages for each account.



## Method 2: Advanced Setup with Cloudflare Worker

GitHub disables scheduled workflows on repositories that have no activity for 60-90 days. To prevent this, you can use a free Cloudflare Worker to trigger the workflow on a schedule, which counts as repository activity.

### Step 1: Create a GitHub Personal Access Token (PAT)

The Cloudflare Worker needs permission to trigger the action in your repository.

1.  Go to your GitHub **Settings**.
2.  Scroll down and click on **Developer settings**.
3.  Go to **Personal access tokens** > **Tokens (classic)**.
4.  Click **Generate new token** > **Generate new token (classic)**.
5.  Give it a **Note** (e.g., "Cloudflare Worker for keep-mega-alive").
6.  Set an **Expiration** date (e.g., "No expiration" for a set-and-forget setup).
7.  Under **Select scopes**, check the box for **`workflow`**. This is the only permission needed.
8.  Click **Generate token**.
9.  **Important:** Copy the generated token immediately. You will not be able to see it again.

### Step 2: Create and Configure the Cloudflare Worker

1.  Log in to your [Cloudflare dashboard](https://dash.cloudflare.com/).
2.  Go to **Workers & Pages** from the left sidebar.
3.  Click **Create application**, then select the **Workers** tab and click **Create Worker**.
4.  Give your worker a name (e.g., `keep-mega-alive-trigger`) and click **Deploy**.
5.  Click **Edit code** to open the worker editor.
6.  Delete the existing code and paste the following Javascript code:
    ```javascript
    // worker.js
    export default {
      async scheduled(event, env, ctx) {
        console.log(`Scheduled trigger executing at: ${new Date().toISOString()}`);
        try {
          await triggerGithubWorkflow(env);
          console.log('Scheduled trigger finished successfully.');
        } catch (error) {
          console.error('Scheduled trigger failed:', error);
        }
      },
    };

    async function triggerGithubWorkflow(env) {
      const { GITHUB_TOKEN, GITHUB_OWNER, GITHUB_REPO, WORKFLOW_ID } = env;

      if (!GITHUB_TOKEN || !GITHUB_REPO || !GITHUB_OWNER || !WORKFLOW_ID) {
        throw new Error("Missing required environment variables!");
      }

      const url = `https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPO}/actions/workflows/${WORKFLOW_ID}/dispatches`;

      const response = await fetch(url, {
        method: 'POST',
        headers: {
          'Authorization': `token ${GITHUB_TOKEN}`,
          'Accept': 'application/vnd.github.v3+json',
          'Content-Type': 'application/json',
          'User-Agent': 'Cloudflare-Worker/1.0',
        },
        body: JSON.stringify({
          ref: 'main', // or your default branch
        }),
      });

      if (!response.ok) {
        const errorText = await response.text();
        throw new Error(`GitHub API error: ${response.status} ${response.statusText} - ${errorText}`);
      }
      
      console.log('Workflow triggered successfully.');
      return { success: true };
    }
    ```
7.  Click **Save and Deploy**.

### Step 3: Add Environment Variables and a Cron Trigger

1.  Go back to your worker's overview page.
2.  Navigate to the **Settings** > **Variables** tab.
3.  Under **Environment Variables**, click **Add variable** and add the following four variables:
    *   `GITHUB_OWNER`: Your GitHub username.
    *   `GITHUB_REPO`: The name of your forked repository (e.g., `keep-mega-alive`).
    *   `GITHUB_TOKEN`: The Personal Access Token you created in Step 1. (Check "Encrypt" for this one).
    *   `WORKFLOW_ID`: The name of the workflow file, which is `my-workflow.yml`.
4.  Now, navigate to the **Settings** > **Triggers** tab.
5.  Under **Cron Triggers**, click **Add Cron Trigger**.
6.  Set the schedule. To run it once a week on Sunday, you can use the cron expression `0 0 * * 0`.
7.  Click **Add Trigger**.

Your setup is now complete. The Cloudflare Worker will now trigger your GitHub Action weekly, ensuring your repository stays active and your MEGA accounts never get deactivated.
