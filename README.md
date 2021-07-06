# SAP on HANA Stop/Start Document and Operations framework


## Versions

Version | Author | Date   | Description   
--|---|---|--
  1.0|N.Olver   |  29 June 2021 | Initial Version, HANA single or distributed.  
  |   |   |  



## Description

This CloudFormation Template will deploy the following elements to enable a step by step stop or start of an SAP application and the EC2 instances on which it is hosted.

For more details on its usage see the blog.


-  **SSM Automation Document (Runbook)**

    The Runbook takes the parameters SAP SID and an action (Stop or Start), the AWS systems manager Runbook will run through a series of steps to Stop or Start the SAP application, Hana Database and EC2 Instances. The supported architectures cover single instance, semi distributed or distributed running HANA in a single tenant scale-up deployment.

- **EventBridge Rules for Document Completion and Step Failure**

   Eventbridge rules are created to push details of success or failure to the SNS topic on completion of the automation document as well as for a specific step which fails. The notification output is parsed to form a link which will take you directly to the automation document contents.

- **SNS Topic**

    An SNS topic is created as a recipient for EventBridge Rules. The CloudFormation document requires an email address as the first subscription. Additional emails can then be added.

- **A CloudWatch Logs Log Group**

    This is not created by the Cloudformation, but defined as a destination for runCommand logs to ensure these are not truncated by any character limit of Systems Manager or deleted as part of housekeeping. Consider adjusting the retention on this group.

:no_entry_sign:  **Not created**

This CloudFormation does NOT create an IAM Role - If you are calling the SSM document from external sources, ensure you build an IAM role with least privilege


## Installation


1. In CloudFormation, select Create Stack and populate the required parameters or leave them as the defaults, ensuring that they are unique in your account. You will need a unique name and an email address.
Once complete. Select _Next_, then under configure stack options select _Next_, review the inputs and select _Create Stack_.
Note: If you are redeploying this template, consider deleting old stacks.

<p align="center">
<img src="images/image-specify-stack-details2.png" alt="image-specify-stack-details" width=80%>
</p>




2. Subscribe to the email which should arrive in your inbox to receive notifications.

<p align="center">
<img src="/images/image-email-subscription.png" alt="image-email-subscription" width=80% >
</p>


3. Check all instances are configured as targets for for [Systems Manager Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-setting-up.html) on EC2 instances including the appropriate instance roles.

4. Setup tags to enable and identify the instances, according to the naming standard defined when you created the stack. Note: the Cloudformation is setup to allow the definition of a different set of tags, but using a prefix like “ssmsap:” clearly identifies a purpose for the tags and will reduce the likelihood of unrelated changes.

<p align="center">
<img src="/images/image-tags-from-cfn.png" alt="image-tags-from-cfn" width=80% >
</p>

    These tags can be added manually to the EC2 Instance tab in the console, or incorporated in your infrastructure deployment templates. Note: The SID of the HANA system (e.g HDB) is not required. Use the Netweaver SID as the HanaTenant value to link the components together. See the diagram for an example. Take care as any typos may cause errors.

    The tag requirements are as follows if using the default tag names. Otherwise adjust to match the values entered during deployment.
    - NOTE: Tag filters are case sensitive, the tag keys are as entered, tag values in uppercase.
        - (all) ** ssmsap:enabled  = ** TRUE (or false to disable)
        - (all) ** ssmsap:role = ** HANA,APP,SCS or combination (if app and scs are on the same instance use APP,SCS all in one use HANA,SCS,APP (order not critical))
        - (on the app and scs) ** ssmsap:sid =  ** The SID of the Netweaver system.
        - (on the hana system) ** ssmsap:hanatenant = ** The SID of the Netweaver system (not the HANA SID unless its the same). Use SAP SID even if your actual db tenant name is not aligned with SID.

<p align="center">
<img src="/images/image-required-tags.png" alt="image-required-tags" width=80% >
</p>


## Usage


:warning: Warning :  This document has the potential to STOP critical systems, including the EC2 instances. Make sure you have the right system. Make sure users are logged out or aware of the shutdown.

:warning: Warning : The document will START or STOP all systems which meet the tag criteria, use the ssmsap:enabled (or equivalent) to exclude systems which you do not want impacted.


Under Systems Manager > Automation. Select Execute Automation and find the document with the name you specified under “Owned by me”. Familiarise yourself with the document by reading through the document and step descriptions.

<p align="center">
<img src="images/image-ssm-doc-descriptions.png" alt="image_ssm_doc_descriptions" width=80% >
</p>

Execute the automation document. We suggest starting with a simple execution, but it is also possible to run parallel documents for multiple SIDs using the rate control option.

Select a role which can be used to execute the automation, or leave blank to execute as the current role. Select an operation and the SAP SID (not the HANA SID) of the system you wish to stop or start.


<p align="center">
<img src="images/image-enter-parameters.png" alt="image_ssm_doc_descriptions" width=80% >
</p>

Monitor the progress. Note: not all steps are run (STOP steps for stop, START steps for start). The execution may expand over multiple pages. The overall status section will let you know whether the document is finished. You should also receive an email with a link back to this section in the console. Select the Step ID for more details on the error (if applicable).

<p align="center">
<img src="images/image-step-progress.png" alt="image_step_progress" width=80% >
</p>



Check SAP connectivity or Instance Status to ensure the Runbook status matches with that of the SAP system and that it has completed successfully.


## Troubleshooting

**Cannot Find Value for InstanceId(s)** - This is almost always related to incorrectly defined tags. If you are cutting and pasting to create the tags, check for spaces / whitespace in either the tag name or value as this will make the selection fail.

<p align="center">
<img src="images/image-input-validation-failure.png" alt="image_input_validation" width=80% >
</p>




***
