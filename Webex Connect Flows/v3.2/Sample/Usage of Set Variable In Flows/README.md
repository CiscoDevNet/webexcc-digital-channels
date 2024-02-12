

# Set Variable Sample Flows Overview :
- This example demonstrates the usage of Set Variable node using Live Chat Flow and Task Routed Flow. However, the use case is not limited to the livechat only. Any workflow can use the Set Variable node.

## Set Variable
- The example uses fields from a pre-chat form to inject into the system, however Set Variable Node can inject any variable into the system.
- For more details on Set Variable please refer to https://help.imiconnect.io/docs/set-variable. 

The folder includes the following sample flows :
## Media Specific Workflow :
- ### Live Chat Inbound Sample Flow with Set Variable :
    - Every inbound customer message sent over the configured customer chat widget will trigger this workflow.
    - This workflow represents a typical use case for a bank, where a customer reaches out to customer support executives for issues with credit cards, debit cards, savings accounts, or loan enquiry.
    - The customer selects the issue type from the pre-chat form dropdown.
    - The customer can also provide a short description of the issue.
    - These additional contextual details about the interaction are injected into the contact center using the "Set Variable" Node. The sample flow gives an example of how to do that using the "Set Flow Variable" method in the "Set Variable" Node.
    - The Agent can view the injected details in the interaction panel of Agent Desktop.
  
      <br><img width="800" alt="Screen Pop" src="../../images/SetVariable.png"><br>

## Media Agnostic Event Workflow :
- ### Task Routed Workflow :
    - The contextual information injected from the media-specific workflows is also available to the shared flow.
    - An "Evaluate" node can extract these details. The sample flow gives an example of how to do that.
   
      <br><img width="800" alt="Helper method" src="../../images/Helper method.png"><br>

    - In this sample flow we can see that in the Live Chat Inbound Flow we have set the IssueDescription and IssueType.And we have marked these fields as agent viewable so agent will be able to view these variables. Also they are marked as agent editable so agent can also edit these variables. We are getting the values of IssueDescription
    and IssueType from the customer entered values of IssueDescription and IssueType. We are extracting these values in the Task Routed Flow. We can see these values
    in the interaction panel on the agent desktop.

      <br><img width="800" alt="Helper method" src="../../images/chat.png"><br>
  
      <br><img width="800" alt="Helper method" src="../../images/SetVariableExample.png"><br>


## How to setup Workflows?
1. Import the flows in your Webex Connect Service.
2. Create a chat widget template like the one shown in this picture.<br><img width="1266" alt="Screenshot 2022-07-14 at 5 28 57 PM" src="https://user-images.githubusercontent.com/109220058/178982103-69f0fefe-d437-485e-b9ef-5355d034fab4.png"><br>
   Use this template in the Form Template field in the Pre-chat form and Receive Node.
3. Select the appropriate queue in the "Queue Task" node for the channel.
4. Make the flow live with the configured asset.

## Note
- The sample flow uses Set Flow Variable method in Set Variable node to inject the variables. The variables can also be injected into the system using Set Global Variable method.
- For using Set Global Variable method:
  - Create a Global Variable on Webex Contact Center Management Portal and save it.
  - In Set Variable Node with the method name Set Global Variable, select that variable and set a value. 