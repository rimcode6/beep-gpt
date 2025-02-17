# BeepGPT - Intelligent notifications powered by Kaskada

BeepGPT keeps you in the loop without disturbing your focus.
Its personalized, intelligent AI continuously monitors your Slack workspace, alerting you to important conversations and freeing you to concentrate on what’s most important.

BeepGPT reads the full history of your (public) Slack workspace and trains a Generative AI model to predict when you need to engage with a conversation.
This training process gives the AI a deep understanding of your interests, expertise, and relationships.
Using this understanding, BeepGPT watches conversations in real-time and notifies you when an important conversation is happening without you.
With BeepGPT you can focus on getting things done without worrying about missing out.

This repo provides a notebook for you to train your own model using your Slack data, as well as a script to run the alerting bot in production.

### Things to do
* To just experiment with Kaskada, feel free to use the [Example Slack Export](#example-slack-export) included in the repo.
* To also experiment with training a model, you will need a OpenAI API key. See [Getting an OpenAI API key](#getting-an-openai-api-key)
* To train a model on your Slack, you will need a json export of your Slack History. An export can be initiated here: https://<your-slack-workspace>.slack.com/services/export. An Admin-level user of your Slack workspace will need to do the export.
* To run the *"Production"* code and receive alerts from a bot, you will need to create a Slack App. See [Creating a Slack App](#creating-a-slack-app)

### Repo Contents

#### Example Slack Export

* `slack-export` contains an example Slack workspace export. To learn more about the format of the export, see: https://slack.com/help/articles/220556107-How-to-read-Slack-data-exports

Note that some PII data has been removed from the export, but it doesn't effect the files for our use case.

#### Model Training Files

* `FineTuning_v2.ipynb` is a Jupyter notebook which contains all the details of how we successfully trained a model to power BeepGPT.
* `human.py` is a python script used in the training process. See section 2.1 in the `FineTuning.ipynb` notebook for more info.
* `FineTuning_v1.ipynb` is an earlier version of the training procedure. Models trained with this notebook don't generalize as well as those trained with the "v2" notebook.

##### Training outputs

* `messages.parquet` is a single file that contains the full history of the slack export, but only includes the fields that we use in the model. See section 1.1 in the notebook to see how it is generated.
* `labels_.json` is a list of userIds that is generated in section 1.3.2 of the notebook. This list is used in the *"Production"* code to convert from an integer token back to a specific user.

#### *Production* code

* `beep-gpt.py` contains the code that watches Slack in real-time and alerts you about important conversations. This code uses `messages.parquet` and `labels_.json` as inputs. Note that this code is not production-ready, but functions well enough to demo the full application path.
    1. To run this code, first make sure you using at least Python 3.8. (3.11 recommended):
    1. Next install the required libraries
        ```
        pip install -r requirements.txt
        ```
    1. Then set the following environment variables:
        * `OPEN_AI_KEY` -> Found here: https://platform.openai.com/account/api-keys
        * `SLACK_APP_TOKEN` -> Found here: https://api.slack.com/apps/<your-app-id>/general, should start with `xapp-`
        * `SLACK_BOT_TOKEN` -> Found here: https://api.slack.com/apps/<your-app-id>/oauth, should start with `xoxb-`
    1. Finally start the script
        ```
        python beep-gpt.py
        ```

* `manifest.yaml` contains a template for creating a new App in Slack

### Getting an OpenAI API key

In order to experiment with training a model, you will need an OpenAI API key.

If you don't yet have an OpenAI account, go here to sign-up: https://platform.openai.com/signup?launch

After signing up, you will need to add billing details in order to obtain an API key. After doing so, you can create a key here: https://platform.openai.com/account/api-keys

### Creating a Slack App

If you want to run the *Production* code, you will need to create a Slack App and install it into a Slack workspace that you have access to.

1. Start here: https://api.slack.com/apps, and click `Create New App`. Choose `From an App Manifest`.
1. Choose the workspace to install the App in.
1. Copy the contents of `manifest.yaml` and paste it into the window. (make sure to paste it in the `yaml` section)
1. Click `Next`, then `Create`
1. Then on the `Basic Information` page, click `Install to Workspace` and follow the auth flow.
1. Finally, under `App-Level Tokens`, click `Generate Tokens and Scopes`. Add the `connections:write` scope, name it `SocketToken`, and click `Generate`. Don't worry about saving the token somewhere safe, you can always re-access it.

After creating the Slack App, any user that wants to be notified from the App needs to first add it to their personal Apps list. Additionally the App needs to be manually added to any channel you want it to watch.

To add the App to your list: In Slack, in the sidebar, goto `Apps` -> `Manage` -> `Browse Apps`. Click on `BeepGPT` to add it to your app list.

To add the App to a channel: In Slack, go to the channel, click the `v` next to the channel name, goto `Integrations` -> `Apps` -> `Add Apps`. Add `BeepGPT`
