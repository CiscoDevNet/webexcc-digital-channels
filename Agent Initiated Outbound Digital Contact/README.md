# Agent Initiated Outbound Digital Contact
## Overview
This feature will bring in the Digital Channels Outbound capability for SMS and Email channels. WebexCC agents will be able to initiate an outbound SMS and Email conversation from WxCC Agent Desktop with a Digital Outbound Widget.

For Early Access, the Digital Outbound Widget can be integrated to agent desktop for specific teams, using `Desktop Layouts` option on `Control Hub`.

## Desktop Layout Overview
The [Desktop Layout](https://help.webex.com/en-us/article/n5595zd/Webex-Contact-Center-Setup-and-Administration-Guide#topic_8230815F4023699032326F948C3F1495) feature allows WebexCC organizations to configure the Webex Contact Center Desktop as per their business requirements. You can customize elements such as logo, title, and widgets. For the complete list of elements that you can customize, see [Define a Custom Desktop Layout](https://help.webex.com/en-us/article/n5595zd/Webex-Contact-Center-Setup-and-Administration-Guide#topic_BF0EBDF65DCB0A552164D6306657C892). You can create a desktop layout and assign it to a team on Control Hub. This layout defines the agent experience on the desktop for all agents who sign in as part of that team.

There are two types of layouts:

- Global Layout: This layout is a system-generated layout that gets assigned by default when you create a team. For more information, see [Create a team](https://help.webex.com/en-us/article/n5595zd/Webex-Contact-Center-Setup-and-Administration-Guide#task_3F807231F17829A02F2FE4993AD171D6). When you create a team, the Global Layout is automatically set as the desktop layout for the team. You cannot delete this layout.


- Custom Layout: A layout that provides a customized desktop experience. You can create a custom layout for one or more teams.

#### Note - If you assign a new desktop layout when an agent is signed in, the agent must reload the page to see the new layout.

The Webex Contact Center Desktop supports three personas:

- Agent

- Supervisor

- Supervisor and an Agent

The JSON layout file has separate sections for each of the personas. The administrator should configure the settings for each persona in the corresponding section of the JSON layout file. For more information about a sample JSON layout file, see [JSON Layout Top-Level Properties](https://help.webex.com/en-us/article/n5595zd/Webex-Contact-Center-Setup-and-Administration-Guide#Cisco_Generic_Topic.dita_a64e92de-79de-4f9c-9a8a-4c5a8cfe9c95).

When Cisco adds a new feature to the Desktop Layout, the unmodified layout is updated automatically with the new features. The updated desktop layout is automatically available to the existing teams that use the unmodified desktop layout. The Desktop users using the unmodified desktop layout receive the new layout-based features when they sign in or reload the browser.

If you are using the Default Desktop Layout.json file without any modification, then it is considered an unmodified layout. However, if you download the Default Desktop Layout.json file and upload it again, it is considered a modified layout even if the file content or filename is not modified.



## Agent Initiated Outbound Desktop Layout
The `Agent Initiated Digital Outbound widget` is not part of the Default Desktop Layout, hence a Custom Desktop Layout is attached [here.](DigitalOutboundDesktopLayout.json) This can be used as-is and configured for for one or more teams to provide the agents access to Digital Outbound Widget. This layout is built on top of the Default Desktop Layout.json.

## Steps to Configure Agent Initiated Outbound  Desktop Layout


1) Login to control hub and select `Contact Center` option from the menu.

2) Under the Contact Center page select the `Desktop Layouts` option under the `Desktop Experience` Section.
3) In the Desktop Layouts page click the button `Create Desktop Layout`
   <br><img width="800" alt="Settings" src="images/Desktop Layout Page.png"><br>
4) Fill the values in the `Layout Details` section 
   - Enter the name and description for the new layout
   - Choose the teams for which you wish to apply the new layout.
   - Under the `JSON File` section choose the option `replace file` and choose the custom Desktop Layout created.
   - Click the option `Create` to apply the Desktop Layout.
     <br><img width="800" alt="Settings" src="images/Layout Details Section.png"><br>
5) Once the Desktop Layout is created, login to the agent desktop or refresh the same if already logged in to see the `Digital Outbound Widget.`
   - Sms Digital Outbound Widget
   <br><img width="800" alt="Settings" src="images/Sms Outbound Widget.png"><br>
   - Email Digital Outbound Widget
   <br><img width="800" alt="Settings" src="images/Email Outbound Widget.png"><br>
     
## Steps to Configure Desktop Layout if Custom Layout is Already Being Used
If you are using Custom Layout, the following section of the layout json can be placed under "advancedHeader" section in "agent" and "supervisorAgent" sections of your custom layout. You can look at the custom layout provided [here for reference.](DigitalOutboundDesktopLayout.json) <Upload the final layout on git>


```
{
    "comp": "digital-outbound",
    "script": "https://wc.imiengage.io/AIC/engage_aic.js",
    "attributes": {
        "darkmode": "$STORE.app.darkMode",
        "accessToken": "$STORE.auth.accessToken",
        "orgId": "$STORE.agent.orgId",
        "dataCenter": "$STORE.app.datacenter",
        "emailCount": "$STORE.agent.channels.emailCount",
        "socialCount": "$STORE.agent.channels.socialCount"
    }
}
```