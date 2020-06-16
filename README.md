# Customer Service Support Technician Tech Test
Test for CS Support candidates

## Requirements

Allowing for setting up the project, this tech test should take around 2 hours to complete.

1. Find the cause of issues in the application:
* There are three bugs to find
* You are not required to *fix* the bugs, your aim is to __identify__ what is causing them

2. Document your process:
  * What did you notice/find?
  * What are your recommendations?

## Getting Started
These instructions will get the application up and running on your local machine for bug finding purposes.

* Install Ruby [Ways of Installing Ruby](https://www.ruby-lang.org/en/downloads)
* Install Yarn [Ways of Installing Yarn](https://yarnpkg.com/lang/en/docs/install)
* Clone this git repository `git clone git@github.com:smartpension/smart-cs-support-test.git`
* cd into the locally cloned repo `smart-cs-support-test`
* Run `bin/setup` 
* Start up your web-server (see the output of `bin/setup`)
 * If you get an error with: `getaddrinfo: nodename nor servname provided, or not known (SocketError)` 
   * Try running your server with`./bin/rails server -b 0.0.0.0`
     
* Navigate to: http://localhost:3000

Remember to document __how__ you identified the bugs and attach your findings to your email back to us, have fun!!


# My Approach

- Firstly I decided to see if any tests were failing and noticed there were no tests accompanying the codebase.
- I then decided to find the bugs by interacting with the browser as a user.

## Bug Number 1 
Page: /companies  
Action: clicking on "Show"

- When clicking on the "Show" button for any company provided in the "/companies" view I was always redirected to "Mickeys Plaice".
- I then looked at the server log and saw that there were two SQL queries and the last one would always show the details for ["company_id", 1] regardless of what company's "Show" button I was clicking (I had made sure the parameters for the company were changing for the corresponding company).
- I then had a look at the companies_controller and focused in particular on the show method (lines 10 and 11).

### Recommendation
-  Remove line 11 ```@company = Company.first``` from the show method in "app/controllers/companies_controller.rb"

## Bug Number 2
Page: /companies/:id/employees/new  
Action: Fill in "Forename" & "Surname" and click "Save"

- When adding a new employee for a company you are prompted to insert values for "Forename" and "Surname" and after clicking the button "Save" an error message appears saying "Surname can't be blank".
- After looking at the server log parameters it is clear that the label surname has been incorrectly named: 
```{"authenticity_token"=>"LxRW7uxjO8s5Gmeyq2t8iOcgF4Gss5lz7+xP/iToX1CkpFbDmV3CTER+eXbnQIHSJSsJ9C0c+ZHTO3FhbaGIfg==", "employee"=>{"forename"=>"charlie", "middlename"=>"brown"}, "company_id"=>"1"}```
- I also found that in the employee model that :forename, :surname and :middlename need to be present and there is no input for surname in the form.

### Recommendation
Current Code in app/views/employees/new.html.erb

```
<p>
  <%= form.label :surname %>
  <%= form.text_field :middlename, class: "form-control" %>
</p>
```

My solution: to include all three required fields

```
<%= form.label :forename %>
<%= form.text_field :forename, class: "form-control" %>

<p>
  <%= form.label :middlename %>
  <%= form.text_field :middlename, class: "form-control" %>
</p>

<p>
  <%= form.label :surname %>
  <%= form.text_field :surname, class: "form-control" %>
</p>
```

## Bug Number 3 
Page: /companies/:id  
Action: Clicking "Edit" for an employee

- While trying to edit an employee record. The companies id in the url would always change instead of the actual user id. For example, trying to edit the 2nd employee for the 1st company results in `/companies/2/employees/1/edit` when it should be `/companies/1/employees/2/edit`
- Checking the server log confirms the params are not changing in the controller.
- The first place I looked is the url for the button on the employees page, upon inspecting "Edit" button the href points to `href="/companies/2/employees/1/edit"`.
- I then went to the corresponding controller and found that there was only one parameter (employee) instead of two.

### Recommendation
Current code in app/views/companies/show.html.erb

```
edit_company_employee_path(employee)
```

My solution:
```
edit_company_employee_path(@company, employee)
```

### Further Recommendations 
- Other observations: Two views for listing employees (EmployeesController#index & CompaniesController#show)
- Add feature and unit tests
- Have a link or button back to list of companies once redirected to the employees.
