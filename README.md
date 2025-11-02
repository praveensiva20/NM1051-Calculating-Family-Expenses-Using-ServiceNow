# NM1051-Calculating-Family-Expenses-Using-ServiceNow
Automates tracking and analysis of household expenses using ServiceNow workflows, enhancing budgeting and financial visibility for families.
Calculating Family Expenses (ServiceNow)
Project overview

A compact guide to implement a family expense tracking solution on ServiceNow. The app provides two tables — Family Expenses (summary per date) and Daily Expenses (individual expense entries) — linked by a relationship. Features include auto-numbering, mandatory fields, read-only fields, related lists, and a Business Rule to aggregate daily expenses into the Family Expenses summary.

Prerequisites

ServiceNow developer account: https://developer.servicenow.com

A provisioned Personal Developer Instance (PDI).

Basic admin access to the instance (to create tables, update sets, business rules, relationships, and form configuration).

High-level steps

Create and select an Update Set.

Create the Family Expenses table and its fields.

Configure auto-numbering for Family Expenses.

Design and set field-level properties (mandatory, read-only).

Create the Daily Expenses table and its fields.

Configure auto-numbering for Daily Expenses.

Create a Relationship between the two tables and add a related list.

Implement a Business Rule on Daily Expenses to update/insert Family Expenses records.

Test with sample data and validate aggregation and UI behavior.

Step-by-step implementation
1) Set up a ServiceNow instance

Sign up or sign in at ServiceNow Developer portal.

Request a Personal Developer Instance (PDI) and wait for the email with credentials.

Log in to your PDI.

2) Create a new Update Set

Go to All → Filter: Local Update Set → New.

Name: Family Expenses → Submit → Make current.

3) Create the Family Expenses table

Go to All → Filter: Tables → New.

Label: Family Expenses.

New menu name: Family Expenditure.

Save the table.

Add fields (columns)

Number — Type: String

Date — Type: Date (mandatory)

Amount — Type: Integer

Expense Details — Type: String (Max length 800)

Save after adding fields.

4) Make the Number field auto-numbered

Open the Number field → Advanced view.

Set Use dynamic default to checked.

Dynamic default value: Get Next Padded Number → Update.

Configure Number Maintenance: All → Filter: Number Maintenance → New.

Table: Family Expenses

Prefix: MFE

Submit

5) Configure the Family Expenses form

All → Filter: Family Expenses → Open.

Click New → Header → Configure → Form Design.

Drag/drop fields to desired layout.

Make Number read-only; make Date, Amount mandatory.

Save the form design.

6) Create the Daily Expenses table

All → Filter: Tables → New.

Label: Daily Expenses. Add module to menu: Family Expenditure.

Save.

Add fields

Number — Type: String

Date — Type: Date (mandatory)

Expense — Type: Integer

Family Member Name — Type: Reference

Comments — Type: String (Max length 800)

Save after adding fields.

7) Auto-number Daily Expenses (same approach)

Set dynamic default to Get Next Padded Number and create Number Maintenance with desired prefix.

8) Configure Daily Expenses form

Form Design: make Number read-only; make Date and Family Member Name mandatory; save.

9) Create relationship between tables

All → Filter: Relationships → New.

Name: Daily Expenses

Applies to table: Family Expenses

Daily Expenses: select Daily Expenses

Save.

10) Add related list on Family Expenses

Open Family Expenses → Header → Configure → Related Lists.

Add Daily Expenses to the selected area → Save.

11) Business Rule — aggregate Daily Expenses into Family Expenses

All → Filter: Business Rules → System Definition → New.

Name: Family Expenses BR

Table: Daily Expenses

Check Advanced

When to run: Insert and Update

Paste the server-side script in the Advanced script area:

(function executeRule(current, previous /*null when async*/) {
  var FamilyExpenses = new GlideRecord('u_family_expenses');
  FamilyExpenses.addQuery('u_date', current.u_date);
  FamilyExpenses.query();


  if (FamilyExpenses.next()) {
    FamilyExpenses.u_amount += current.u_expense;
    FamilyExpenses.u_expense_details += ">" + current.u_comments + ":" + "Rs." + current.u_expense + "/-";
    FamilyExpenses.update();
  } else {
    var NewFamilyExpenses = new GlideRecord('u_family_expenses');
    NewFamilyExpenses.u_date = current.u_date;
    NewFamilyExpenses.u_amount = current.u_expense;
    NewFamilyExpenses.u_expense_details = ">" + current.u_comments + ":" + "Rs." + current.u_expense + "/-";
    NewFamilyExpenses.insert();
  }
})(current, previous);

Note: Ensure field names (u_date, u_expense, u_comments, u_family_expenses) match your actual table column names. Adjust if ServiceNow auto-generated different u_ prefixes.

12) Refine relationship query (optional)

Open the Relationship record and set a query script similar to:

(function refineQuery(current, parent) {
  current.addQuery('u_date', parent.u_date);
  current.query();
})(current, parent);

Save/Update.

Testing checklist

Create multiple Daily Expenses entries with the same date and verify the Family Expenses record for that date updates the amount and details correctly.

Create a Daily Expense with a new date and verify a new Family Expenses record is created.

Validate related list displays Daily Expenses under Family Expenses.

Confirm read-only and mandatory behaviors on the forms.

Troubleshooting & tips

If auto-numbering doesn’t work, re-check Number Maintenance entry and dynamic default on the field.

Verify Business Rule field names if aggregation fails.

Use server logs and gs.info() for debugging in Business Rules.

Next steps / enhancements

Add categories, tags, or budgets per family member.

Build reports and scheduled email summaries.

Add security ACLs for data privacy per user or role.

Create a UI page/dashboard for visual charts.
