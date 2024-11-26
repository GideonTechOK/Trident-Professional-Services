# Creating a Multi-Level Approval Workflow in SharePoint: An Alternative to Power Automate Sequential Approvals

When managing complex approval processes, a **multi-level approval workflow** can streamline communication and ensure clear tracking of requests. While Power Automate provides robust sequential approval functionality, SharePoint lists can also be used to create an effective multi-level approval system. This approach is particularly useful when you need a simple solution without relying entirely on Power Automate.

In this guide, we'll walk through how to create a multi-level approval system using **SharePoint**, along with **Power Automate** for advanced functionality when needed.

---

## Why Use SharePoint for Multi-Level Approvals?

SharePoint provides:
1. **Centralized Data Management**: Store request and approval details in lists.
2. **Custom Columns for Tracking**: Use columns to represent stages of approval and comments.
3. **Integration with Power Automate**: Automate parts of the workflow while keeping the data in SharePoint.

---

## Setting Up the SharePoint Lists

### 1. **Create an "Approvers" List**
   - **Purpose**: Store approvers and their roles.
   - **Steps**:
     1. Go to your SharePoint site and create a new list named **Approvers**.
     2. Add the following columns:
        - **Title** (Single Line Text): Use this to define the stage (e.g., "Level 1", "Level 2").
        - **Approver** (Person or Group): Assign the approver for each level.
     3. Populate this list with approvers for each stage.

### 2. **Create a "Requests" List**
   - **Purpose**: Manage user requests and their approval status.
   - **Steps**:
     1. Create a list named **Requests**.
     2. Add the following columns:
        - **Title** (Single Line Text): The request title.
        - **Approval Status** (Choice): Options like "Pending", "Approved", "Rejected".
        - **Comments** (Multi-line Text): For approver feedback.
        - **Approver** (Single Line Text): To track who is currently reviewing.
        - **WorkstationType** (Choice): Add options like "Standard", "Performance", "2-in-1".

---

## Setting Up the Multi-Level Approval Workflow in Power Automate

While SharePoint tracks the approval process, Power Automate can handle notifications and status updates.

### **Flow Overview**

1. **Trigger**: When a new item is created in the **Requests** list.
2. **Get Approvers**: Retrieve the list of approvers for each stage.
3. **Send Approvals**: Sequentially send approval requests to each approver.
4. **Update Status**: Update the **Approval Status** and **Comments** in the **Requests** list.

---

### Step-by-Step Power Automate Configuration

1. **Trigger the Flow**:
   - Select **When an item is created** from the **SharePoint** connector.
   - Configure the trigger to monitor the **Requests** list.

2. **Retrieve Approvers**:
   - Use the **Get Items** action to fetch approvers from the **Approvers** list.
   - Filter the approvers based on the current approval stage.

3. **Initialize Variables**:
   - **ApprovalStatus** (String): Initialize with "Pending".
   - **Comments** (String): Initialize as blank.

4. **Loop Through Approvers**:
   - Add an **Apply to Each** loop to iterate through the retrieved approvers.
   - Inside the loop:
     - Use the **Start and Wait for an Approval** action:
       - Set approval type to **Approve/Reject - First to Respond**.
       - Assign the task to the approver from the **Approvers** list.
     - Append approver feedback to the **Comments** variable.
     - Set the **ApprovalStatus** variable based on the approver's response.

5. **Condition for Status Update**:
   - Add a condition to check if the **ApprovalStatus** is "Rejected/Cancelled".
     - **Yes**: Update the item in SharePoint with "Rejected" status and comments.
     - **No**: Continue to the next approver.

6. **Update SharePoint List**:
   - Once all approvers have responded, update the **Approval Status** in the **Requests** list.

---

## Testing the Workflow

1. **Add Test Data**:
   - Create a new item in the **Requests** list with all necessary details.
2. **Monitor the Flow**:
   - Use the **Power Automate** run history to ensure that each step executes correctly.
3. **Verify Updates**:
   - Check that the **Requests** list reflects the correct approval status and comments.

---

## Advantages of This Approach

1. **Flexibility**: Modify approval stages and roles directly in the SharePoint list without changing the flow.
2. **Scalability**: Handle multiple approvals simultaneously by configuring parallel branches in Power Automate.
3. **Centralized Data**: Maintain a clear history of approvals and comments in the SharePoint list.

---

## Conclusion

Using SharePoint for multi-level approvals provides a simple yet powerful alternative to Power Automate's sequential approvals. By combining SharePoint's list capabilities with Power Automate's automation, you can create a system that tracks, notifies, and manages approvals efficiently.

Start small with a basic configuration, then scale by adding more stages, parallel approvals, or advanced logic. This approach ensures that your workflows remain dynamic, transparent, and adaptable to changing business needs.

---

*Written by The Ideas Forge Team*

*Tags: #SharePoint #PowerPlatform #Approvals #Automation #Workflow #Microsoft365*
