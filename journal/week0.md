# Week 0 — Billing and Architecture
Contains an outline of all the tasks have been able to submit
## Required HomeWork/Tasks 
### Install and verify AWS CLI
I installed the AWS CLI using the instructions from the outline.
![AWS CLI - working](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/612f22eb41bc4c03cc6d89877ed884a2ed7a68d2/journal/assets/Week0-Working%20AWSCLI%20proof.png)

### Create alarms
I created alarms by following the below steps
#### Create an SNS topic using CLI
I created the SNS topic using AWS CLI 

#### Create Billing alarms using CLI
I created a billing alarm that measures the DailyEstimatedCharges. I put the threshhold to $1 dollar to ensure I am able to track my actual spending. 
![Daily Billing ALarm](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/612f22eb41bc4c03cc6d89877ed884a2ed7a68d2/journal/assets/Week0-Billing%20Alarms.png)
### create Budgets
I worked on the budgets bit using aws console, and created a zero spend budget to ensure that I get a notification once we start using paid services. My budget is to spend a maximum of $20 per month in this project. My total budget spend for the whole program is $60. I will create an extra budget alarm as the program progress.
![Zero spend budget](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/612f22eb41bc4c03cc6d89877ed884a2ed7a68d2/journal/assets/Wek0-%20Zero%20Spend%20Budgets.png)
### Cruddur Architectural Diagram
Cruddur Architecture Diagram drawn out using Lucid chart
![Cruddur Architecture Diagram](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/612f22eb41bc4c03cc6d89877ed884a2ed7a68d2/journal/assets/Cruddur%20Architecture%20Diagram.png)
[Lucid chart cruddur Architecture Link
](https://lucid.app/lucidchart/e87b5a5f-40a2-479b-ab3f-0466598d1377/edit?viewport_loc=-448%2C-177%2C3382%2C1433%2C0_0&invitationId=inv_918b374e-1f2f-4fab-8479-4ed2161a4f68)
### Knowledge Challenges
I went through the videos on pricing considerations and security consideration. afterwards I completed the quizes
![Proof Quiz challenges Completed](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/612f22eb41bc4c03cc6d89877ed884a2ed7a68d2/journal/assets/Week0-%20Quizs%20Completed.png)
## Homework Challenges

### Cruddur Napkin design
![Cruddur conceptual Idea
](https://github.com/lkiunga/aws-bootcamp-cruddur-2023/blob/d0bdf3295c9645431f9191ef5cd53639bd8e7934/journal/assets/Cruddur%20Napkin%20design.jpeg)
### Documenting Code on Markdown
```
{
    "AlarmName": "DailyEstimatedCharges",
    "AlarmDescription": "This alarm would be triggered if the daily estimated charges exceeds 1$",
    "ActionsEnabled": true,
    "AlarmActions": [
        "arn:aws:sns:us-east-1:997272838836:CB-billing-alarms"
    ],
```

### Creating a TO Do list on Markdown Languages
- [X] Create AWS Admin User.
- [X] ADD MFA to the root user.
- [X] Create an Organizational Unit
- [ ] Enable AWS Cloud Trail
- [ ] Create IAM Roles and Organizational SCP
