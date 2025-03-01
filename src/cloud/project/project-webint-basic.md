---
group: cloud-guide
title: Configure your project
functional_areas:
  - Cloud
  - Configuration
---

For all {{site.data.var.ece}} projects, you can use the [Project Web Interface](https://account.magento.com/customer/account/login/) to perform the following tasks:

-  [Access projects](#project-access)
-  [Configure environment settings](#configure-environment-settings)
-  [Add users and manage access]({{ site.baseurl }}/cloud/project/user-admin.html)
-  [Manage Git branches]({{ site.baseurl }}/cloud/project/project-webint-branch.html)

Changes that you make to environment configuration trigger a redeployment of that environment.

## Access your project and environments {#project-access}

The Project Web Interface provides several ways to access your project and environments:

-  Storefront URL for each active environment
-  Secure Shell (SSH) link for SSH access via terminal application
-  Clone the project using the Magento Cloud CLI or Git

To access projects and environments through the Project Web Interface:

1. [Log in to your project](https://account.magento.com/customer/account/login/).
1. The left pane shows your list of environments.
1. Click **Access Site** for a list of storefront URLs for web access and the command for SSH access.

For more information about using SSH, see [SSH to an environment]({{ site.baseurl }}/cloud/env/environments-ssh.html#magento-cli). To clone the project using either the {{site.data.var.ece}} CLI or Git, use the links in the field under the branch name.

The following figure shows an example.

![Clone the project]({{ site.baseurl }}/common/images/cloud/cloud_project-clone.png){:width="600px"}

Click either **CLI** or **Git** to display the appropriate clone command. Use the ![Copy to clipboard]({{ site.baseurl }}/common/images/cloud/cloud_copy-to-clipboard.png) (Copy to clipboard) button to copy the command to the clipboard.

## Configure environment settings {#project-conf-env-set}

You can set the following configuration options for each environment:

<table>
   <tr>
      <th style="width= 300px;">Option</th>
      <th>Description</th>
   </tr>
   <tr>
      <td>Environment status</td>
      <td>An environment can be either <em>active</em> or <em>inactive</em>. You'll do most of your work in an active environment. After merging an environment with its parent, you can optionally delete the environment, making it inactive. To delete an environment, click <strong>Delete</strong>. You can active an inactive environment later.</td>
   </tr>
   <tr>
      <td>Outgoing emails</td>
      <td>Setting this option <strong>On</strong> enables support for sending emails from the environment using the SMTP protocol.</td>
   </tr>
   <tr>
      <td>HTTP access control</td>
      <td>Setting this option <strong>On</strong> enables you to configure security for the Project Web Interface using a login and also IP address access control.</td>
   </tr>
</table>

### Configure emails for testing {#email}

By default, email support is **disabled** on Staging and Production environments. You must enable email support on an environment to send emails including Welcome, password reset, and two-factor authentication emails for Cloud project users.

You can manage email support for each Cloud project environment from the Project Web Interface or from the command line.

-  On master and integration branches, use the *Outgoing emails* toggle in the Project Web interface to enable or disable email support.

-  On Production and Staging environments or other environments where the *Outgoing emails toggle* is not available in the Project Web Interface, you can check the current configuration from the Project Web Interface, but you can only change the configuration from the command line using the [Cloud CLI for Commerce]({{ site.baseurl }}/cloud/reference/cli-ref-topic.html) `environment:info` command to set the `enable_smtp` property.

   Enabling SMTP updates the `MAGENTO_CLOUD_SMTP_HOST` environment variable with the IP address of the SMTP host for sending mail.

{% include cloud/note-env-config-redeploy-warning.md%}

{:.procedure}
To manage email support from the Project Web Interface:

1. [Access your project](#project-access) and select the environment to configure.

1. Select **Configure environment**, and then select the **Variables** tab.

1. To enable or disable outgoing emails, toggle *Outgoing emails* **On** or **Off**.

   ![Set outgoing emails]({{ site.baseurl }}/common/images/cloud/cloud-env-outgoing-emails-config.png){:width="650px"}

After you change the setting, the environment builds and deploys with the new configuration.

{:.procedure}
To manage email support from the command line:

1. Check the current email configuration in the Project Web Interface.

   ![Check outgoing email configuration]({{ site.baseurl }}/common/images/cloud/cloud-env-outgoing-emails-current-setting.png){:width="650px"}

1. If needed, change the email configuration.

1. From the command line, change to the directory where you [cloned your Cloud project]({{ site.baseurl }}/cloud/before/before-setup-env-2_clone.html#clone-the-project).

1. Log in to the project.

   ```bash
   magento-cloud login
   ```

1. If you do not have the project ID and environment ID for the environment, use the following commands to get the values.

   ```bash
   magento-cloud project:list
   ```

   ```bash
   magento-cloud environments -p <project-id>
   ```

1. Change the email support configuration by setting the `enable_smtp` environment variable to `true` or `false`.

   ```bash
   magento-cloud environment:info --refresh -p <project-id> -e <environment-id> enable_smtp true
   ```

   Wait for the environment to build and deploy.

1. Verify that email is working:

   -  Use SSH to connect to your environment.

   -  Check that the `MAGENTO_CLOUD_SMTP_HOST` value is set.

      ```bash
      printenv MAGENTO_CLOUD_SMTP_HOST
      ```

   -  Send a test email to your email address or another address you can check.

      ```bash
      php -r 'mail("mail@example.com", "test message", "just testing", "From: tester@example.com");'
      ```

## Set project and environment variables {#project-conf-env-var}

You can set project wide and environment specific variables through the Project Web Interface. Variables can be either text or JSON format. _Project variables_ are set across all branches and environments. Use _environment variables_ for sensitive data like payment method information. For more information, see [Environment variables]({{ site.baseurl }}/cloud/env/variables-intro.html).

To view or edit environment variables, you must have at minimum the project reader role with [environment admin]({{ site.baseurl }}/cloud/project/user-admin.html) privileges.

{% include cloud/wings-variables.md %}

### Project variables {#project}

To set project variables in the Project Web Interface:

1. [Access your project](#project-access).
1. Select the Settings icon, and then select the **Variables** tab.

   ![Configure project variables]({{ site.baseurl }}/common/images/cloud/cloud-config-project-variables.png){:width="650px"}

1. Click **Add Variable**.
1. In the **Name** field, enter a variable name. For example, to set the Admin email for the default account, enter `ADMIN_EMAIL`.
1. In the **Value** field, enter the value for the variable. For example, enter a valid email address accessible for reset email notifications.

   ![Set project variables]({{ site.baseurl }}/common/images/cloud/cloud_project_variable.png)

1. As needed, select options for **JSON value**, **Visible during build**, and **Visible during runtime**. If you do not have Super User access, you may only see the JSON value option.
1. Click **Add Variable**. After you add the variable, the environment will deploy. Wait until deployment completes before more edits.

### Environment variables {#env}

To set environment specific variables in the Project Web Interface:

1. [Access your project](#project-access) and select a specific environment.
1. Select **Configure environment**.
1. Select the **Variables** tab.
1. Click **Add Variable**.
1. In the **Name** field, enter a variable name. For example, to set the Admin default account password, enter `ADMIN_PASSWORD`.
1. In the **Value** field, enter the value for the variable. For example, enter a valid email address accessible for reset email notifications.

   ![Set environment variables]({{ site.baseurl }}/common/images/cloud/cloud_env-var.png){:width="650px"}

1. As needed, select options for **JSON value**, **Enabled**, **Inheritable by child environments** and **Sensitive**. If you do not have Super User access, you may only see the JSON value option.
1. Click **Add Variable**. After you add the variable, the environment will deploy. Wait until deployment completes before more edits.

{:.bs-callout-warning}
If you are attempting to [override configuration settings]({{ site.baseurl }}
), you must prepend `env:` to the variable name. For example:
![Environment variable example]({{ site.baseurl }}/common/images/cloud/cloud_env_var_example.png)

## Configure routes {#project-conf-env-route}

Routes allow you to set redirects or upstream settings for applications for your specific environment. For full details on routes, see [routes.yaml]({{ site.baseurl }}/cloud/project/routes.html). These routes (or URLs) are used to access your storefront.

1. [Access your project](#project-access) and select a specific environment.
1. Select **Configure environment**.
1. Select the Routes tab.
1. Select **Add Route**.
1. Enter a URL. You can use `{default}` in the URL, which is a placeholder for the default domain.
1. Select a **Type**: Upstream for applications or Redirect.
1. To configure an Upstream route:

   1. Enter the **Upstream** route.
   1. Use the toggle to enable or disable the **Cache** for the route.
   1. Enter the cookies to list: No cookies, All cookies, or Specify a specific cookie. You can enter multiple specific cookies.
   1. To add Headers to an allowlist, select Default Headers or Specify a header. You can enter multiple headers.
   1. Use the toggle to enable or disable the Server-Side Includes (**SSI**).

1. To configure a Redirect, enter a URL to **Redirect to**. You can use `{default}` in the URL, which is a placeholder for the default domain.
1. Click **Add Route** to save. The setting is saved and deployed to the environment.

   ![Configure a route]({{ site.baseurl }}/common/images/cloud/cloud_routes.png){:width="650px"}

## View environment history {#project-conf-hist}

An environment's history includes:

-  Initial creation
-  Snapshots
-  Syncs and merges
-  Code pushes

To view the history for an environment, log in to your project and select the environment. The page displays a general history of actions completed on the page. For a detailed list of completed actions during build and deployment, Adobe recommends reviewing logs directly on the servers. See [View logs]({{ site.baseurl }}/cloud/project/log-locations.html).

The following figure shows a sample history.

![Sample environment history]({{ site.baseurl }}/common/images/cloud/cloud_environment-history.png){:width="800px"}

The history shows, from oldest to newest:

-  Environment branched from `FeatureX`
-  Environment synced with the parent
-  Environment snapshot created

Adobe recommends [creating a snapshot]({{ site.baseurl }}/cloud/project/project-webint-snap.html) before you make any code changes.

-  Environment variable added
-  Environment snapshot created
