# Instructions for using bot flow templates

## Q&A bot flow for Livechat channel
Pre-requisites
You must configure the following entities:
1.	A Webex Connect tenant with bot builder app, WxCC nodes, and Livechat channel enabled
2.	A Livechat app in connect
      * A website configuration in Webex Engage for the Livechat asset.
      * Pre-chat form template in Webex Connect if you want to use a pre-chat form. The nodes specific to gathering pre-chat form can be deleted from the flow if you choose       to not use a pre-chat form.
3. Q&A bot in the bot builder
    * A Q&A bot with desired articles should be created in the bot builder app. This will create an imibot integration in Webex Connect that can be selected from a dropdown in the Q&A bot node.
    * The Q&A bot should be trained and made live for the bot responses to reach the Livechat widget.
    * Refer to the Webex Connect documentation for steps to create your first Q&A bot.
      https://help.imiconnect.io/docs/qa-bots

## Q&A bot flow for SMS channel
Pre-requisites
You must configure the following entities:
1.	A  Webex Connect tenant with bot builder app, WxCC nodes, and SMS channel enabled
2.	A SMS number in Webex Connect
3.  Q&A bot in the bot builder
    * A Q&A bot with desired articles should be created in the bot builder app. This will create an imibot integration in Webex connect that can be selected from a dropdown in the Q&A bot node.
    * The Q&A bot should be trained and made live for the bot responses to reach the customer over sms channel.
    * Refer to the Webex Connect documentation for steps to create your first Q&A bot
      https://help.imiconnect.io/docs/qa-bots

## Task bot flow for Facebook channel
Pre-requisites
(Optional) This sample flow leverages a google sheet to fetch user info and add that user info to the said google sheet via sheet2api APIs. If you want to emulate this flow, you will need:

1. A publicly accessible google sheet and a sheet2api account that has access to your google sheet. Refer to sheet2api documentation on how to make requests to your sheet.

You can choose to build a task bot flow for a different use case that leverages certain other APIs or does not require fulfillment via external APIs at all. In such cases, you can remove the HTTP Request, Branch, and Data Parser nodes after the ‘Create task’ node and before the ‘Task bot’ node. The Branch node after the Task bot node and the subsequent branches are also use case dependent and will require modification depending on the bot’s use case.

Apart from this, you will require:

1. A Webex Connect tenant with bot builder app, WxCC nodes, and Facebook channel enabled
2.	A Facebook app in connect
    * This will require a Facebook page with a ‘Send message’ button. Once you have your page ready, you can create a Facebook app in the ‘Apps’ screen in connect. Refer to the Webex Connect documentation for more information.
3. Task bot in the bot builder
    * A Task bot with desired intents and response template keys should be created in the bot builder app. This will create an imibot integration in connect that can be selected from a dropdown in the Task bot node.
    * The Task bot should be trained and made live for the bot responses to reach the Facebook page.
    * Refer to the Webex Connect documentation for steps to create your first Task bot - https://help.imiconnect.io/docs/task-bots.
4. For more details on sheet2api APIs, refer to https://sheet2api.com/documentation
