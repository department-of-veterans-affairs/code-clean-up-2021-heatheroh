## Secret scanning

- [Secret scanning](#secret-scanning)
  - [Viewing and managing results](#viewing-and-managing-results)
  - [Introducing a test secret](#introducing-a-test-secret)
  - [Excluding files from secret scanning](#excluding-files-from-secret-scanning)
  - [Managing access to alerts](#managing-access-to-alerts)


### Viewing and managing results
The security tab in the repository navigation bar will indicate that there are new security alerts.  There are multiple ways to view and triage your secret scanning results. Here is one.

1. Navigate to the security page by clicking the tab that reads "Security"
2. To view your Secret alerts, on the left hand side of this page is a navigation menu, click on "Secret Scanning Alerts"
4. To manage your Secret alerts, for each secret, look at the options to close it by 
    - Clicking the checkbox in the upper left-hand corner of each alert 
    - Clicking the "Mark as" dropdown that appears in the upper right corner after selecting a checkbox
    - Choosing an option that is most suitable for the selected alerts

### Introducing a test secret
When developing test cases, secrets are sometimes introduced that cannot be abused when disclosed. Secret scanning will still detect and alert on these secrets.

1. In the GitHub repository find the file named `storage-service/src/main/resources/application.test.properties`
2. Add the following secrets to the file (Don't worry, these are not real secrets :smile:)
        ```
        AWS_ACCESS_KEY_ID="AKIAZBQE345LKPTEAHQD"
        AWS_SECRET_ACCESS_KEY="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"
        ```
3. Watch the results and determine if the secret is detected when the file is stored.
4. How would you like to manage results from test files? There are multiple options for handling the results, you can choose from the following list: 
    - Open
    - Revoked
    - False Positive
    - Used in tests
    - Won't fix

### Excluding files from secret scanning
While we can close a detected secret as being "Used in tests", we can also configure secret scanning to exclude files from being scanned.

1. Create the file `.github/secret_scanning.yml` if it doesn't already exist in the repository.
2. Add a list of paths to exclude from secret scanning. You can use [filter patterns](https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet) to specify paths.
    ```yaml
    paths-ignore:
        - '**/test'
    ```
    **Note**: The characters `*`, `[`, and `!` are special characters in YAML. If you start a pattern with `*`, `[`, or `!`, you must enclose the pattern in single quotes.

    Use a pattern to exclude the file `storage-service/src/main/resources/application.test.properties`

    <details>
    <summary>Solution</summary>
    A possible solution is:

    ```yaml
    paths-ignore:
        - '**/test/**'
        - '**/application.test.properties'
    ```
    </details>

3. Test the pattern by adding another secret to the file `storage-service/src/main/resources/application.test.properties`

    For example add this secret to the file
    ```
    AWS_SECRET_ACCESS_KEY_2="6L=yQr6Ivxxj/XG+YdFPdH/xWDcbSV9ch/EjmHCL"
    ```
4. You should see that the file is now being excluded from secret scanning.

### Managing access to alerts
Due to the nature of secrets, the alerts are only visible to organization owners and repository administrators.

**Note:** Members and/or teams require write privileges on the repository before access to alerts can be given.

To grant access to other members and teams:
  1. Click on the "Settings" tab in the repository's navigation bar
  2. Choose "Security & analysis" from the menu on the left-hand side
  3. Enable Dependabot alerts
  4. In the "Access to alerts" section at the bottom, add members or teams to provide access to your repository's Secret alerts

💡**Now that we're familiar with Secret scanning, let's learn about how to remove sensitive data when it happens to find its way into our repository. [Click here](remove-sensitive-data.md) to move to the next section.** 💡
