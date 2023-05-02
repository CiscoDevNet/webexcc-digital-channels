

# Set Variable Sample Flows Overview : 
  - This example demos the usage of Set Variable Node using Live Chat Inbound Message Workflow and Task Routed Workflow, but any workflow can use the Set Variable Node. Also, the example uses fields from a pre-chat form to inject into the system, but Set Variable Node can inject any variable into the system. The use case is not limited to the example.
  - Please refer to the documentation for more details: https://help.imiconnect.io/docs/set-variable
  
The folder includes the following sample flows for set variable : 
## Media Specific Workflow : 	
- ### Live Chat Inbound Message Workflow :
  - Every inbound customer message sent over the configured customer chat widget will trigger this workflow.
  - This workflow represents a typical use case for a bank, where a customer reaches out to support executives for issues with Credit Card, Debit Card, Savings accounts or Loan Enquiry.
  - The customer selects the issue type from the pre-chat form dropdown.
  - The customer can also provide a short description of the issue.
  - These additional contextual details about the interaction are injected into the contact centre using the "Set Variable" Node. The sample flow gives an example of how to do that using the "Set Flow Variable" method in the "Set Variable" Node.
  - The Agent can view the injected details in the interaction panel of Agent Desktop.
## Media Agnostic Event Workflow :
- ### Task Routed Workflow :  
  - The contextual information injected from the media-specific workflows is also available to the shared flow.
  - An "Evaluate" node can extract these details. The sample flow gives an example of how to do that.
## Quick Start on Workflows
  1. Import the flows in your Webex Connect Service.
  2. Create a chat widget template like the one shown in this picture.<img width="1266" alt="Screenshot 2022-07-14 at 5 28 57 PM" src="https://user-images.githubusercontent.com/109220058/178982103-69f0fefe-d437-485e-b9ef-5355d034fab4.png">
 Use this template in the Form Template Field in the "Pre-chat form" and "Receive" Node.
  3. Select the appropriate queue in the "Queue Task" node for the channel. 
  4. Make the flow live with the configured asset.

## Note
  - The sample flow uses the "Set Flow Variable" method in the "Set Variable" node to inject the variables. The variables can also be injected into the system using the "Set Global Variable" method.
  - For using the "Set Global Variable" method:
    - Create a Global Variable on WxCC Portal and Save it.
    - In the Set Variable Node with the method name "Set Global Variable", select that variable and set a value. 
