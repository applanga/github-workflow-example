# Applanga Github Workflow Integration Example
 
The example [repository](https://github.com/applanga/github-workflow-example) showcases a full cycle [github workflow setup](https://github.com/applanga/setup-applanga-cli) integration with the [Applanga Localization Platform](https://www.applanga.com).

The benefit of using github workflows is that you can automate your localization process without the need to share any repository credentials with your localization provider.

To use github workflows on your repository you need to create a folder called .github/workflows/ and place the workflow configuration .yml files in there. 

For a more detailed introduction to github workflows please see the [Github Documentation](https://help.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow)

---
# Workflow Configurations

The [repository](https://github.com/applanga/github-workflow-example) contains 2 workflows. Running and past workflow actions can be tracked under the [**Actions** Tab](https://github.com/applanga/github-workflow-example/actions).

[.github/workflows/applanga-push.yml](https://github.com/applanga/github-workflow-example/blob/master/.github/workflows/applanga-push.yml) will push any `.json` source files under the directory `react_json_sample/en/` to Applanga whenever they are changed in the repository. Depending on your folder structure and file format you need to modify the `paths` for your repository workflow config.

Note that for the above workflow file the `Applanga Push Action` will only get triggered when there is a push to the `main` branch as specified in the config. It is also possible to specify a different branch or add more branches as needed.

[.github/workflows/applanga-pull.yml](https://github.com/applanga/github-workflow-example/blob/master/.github/workflows/applanga-pull.yml) pulls new translations available from Applanga and then creates a pull request in the repo with the newly added languages or updated translation files. The workflow configuration is configured to run on any given branch. To set this up successfully there are 2 important requirements
1. The workflow file must first be created from the default branch. In other words start first by adding the workflow file to the `master` or `main` branch(whichever is the default) of the repository, which allows github to pick up the workflow. This is a github limitation, and more info can be found on the following github [issue](https://github.community/t/workflow-dispatch-event-not-working/128856/2)
2. A **Webhook Endpoint** has to be configured for the project on Applanga dashboard to trigger the workflow. See [Configure Webhook Endpoint](#configure-webhook-endpoint) below for how to configure 
a webhook endpoint.

Note that if you are already having a project setup on Applanga and you want to test the pull command, all keys that you want to pull should be tagged with `app:translations.json`, because of the current Cli configuration example. For more info see the [Applanga Cli documentation](https://www.applanga.com/docs/integration-documentation/cli).

---
## Configure Webhook Endpoint
To trigger the `applanga-pull` workflow a webhook endpoint has to be configured in the projects settings page on applanga. 

The webhook is triggered at least 15 minutes from when there is no translation change. This means whenever translation is added or edited a webhook request is scheduled to be sent to all configured endpoints for the project 15 minutes later. The scheduled request will be sent as planned unless there is a new change to translation before the scheduled time, in which case it is rescheduled to be sent 15 minutes later.

Here are the steps to setup the **Webhook Endpoint**
* Login to Applanga dashboard and navigate to the project. Then click on **Project Settings**

![](https://www.applanga.com/assets/images/docu/groups_editapp.png)

* In the settings page scroll down to the section **WEB HOOKS** and click the **Add endpoint** button, this will show a modal where the endpoint values can be entered

![](https://www.applanga.com/assets/images/docu/webhook_settings.png)

* Set the http method to *POST* and enter the endpoint url as follows `https://api.github.com/repos/<OWNER>/<REPO>/actions/workflows/applanga-pull.yml/dispatches`. The following values `<OWNER>` and `<REPO>` should be replaced with the correct values.

Please refer to the following screenshot 

![](https://www.applanga.com/assets/images/docu/webhook_branch_trigger_endpoint_url.png)

* **Headers** You need to set an `Authorization` header. The `Authorization` header value is a valid github personal access token with repository permissions combined like so `token <GH_PERSONAL_ACCESS_TOKEN>` where `<GH_PERSONAL_ACCESS_TOKEN>` is the personal access token generated on github. For example if you had an access token `ghp_zWAdtUqmtFTY7qkqJS1wmuEx8ytX0SpIPv` then the Authorization header would be set like so `token ghp_zWAdtUqmtFTY7qkqJS1wmuEx8ytX0SpIPv`. Please refer to the following github [documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) on how to generate an access token.

![](https://www.applanga.com/assets/images/docu/webhook_branch_trigger_headers.png)

* **Body** Click the **Body** tab and select **JSON**. A raw JSON text should be pasted in the textbox that contains the field `ref` which should be set to the name of the branch in which the workflow is intended to be triggered. For example if the workfow should be triggered in a branch named `main` then the text to be pasted would be as follows

```
{
  "ref": "main"
}
```

Next click **Save endpoint**.

![](https://www.applanga.com/assets/images/docu/webhook_branch_trigger_body.png)

For more information about triggering a workflow dispatch event via REST Api check [here](https://docs.github.com/en/rest/actions/workflows#create-a-workflow-dispatch-event).

---
# Applanga Configuration

Because the workflows make use of the [Applanga Command Interface](https://github.com/applanga/applanga-cli) you also need to add a [.applanga.json](https://github.com/applanga/github-workflow-example/blob/master/.applanga.json) configuration file to your repository. 

To uniquely identify your project you need to provide your ***API Access Token*** which can be found on the **API** section in the Applanga Project Settings in the [Applanga Dashboard](https://dashboard.applanga.com). Select your project, click ***Project Settings***, click on **“Show API Token”** and copy the Token. The token can be stored as a [Encrypted Repository Secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets-for-a-repository) labeled **APPLANGA\_ACCESS\_TOKEN** or as plain text directly in the config e.g.: `"app": { "access_token": "<APPLANGA_API_TOKEN>", ... }`.

The included config is set-up to push all changes to `translations.json` file under the directory `react_json_sample/en/` to Applanga and pull all other languages from Applanga as well as create the needed directories on the daily schedule or via the commandline request as described above.

The example configuration is set-up to use the `"react_simple_json"` file format but Applanga supports a wide variety of other file formats and folder structures. Also the example works with only one file per language if you need to support more you need to provide a `tag` per file for details on that and more see the [Applanga CLI Documentation](https://github.com/applanga/applanga-cli).

```json 
{
  "app": { 
    "pull": {
      "target": [
        {
          "file_format": "react_simple_json", 
          "exclude_languages": ["en"],
          "tag": "app:translations.json",
          "path": "./react_json_sample/<language>/translations.json"
        }
      ]
    }, 
    "push": {
      "source": [
        {
          "language": "en",
          "file_format": "react_simple_json", 
          "tag": "app:translations.json",
          "path": "./react_json_sample/en/translations.json"
        }
      ]
    }
  }
}
```
