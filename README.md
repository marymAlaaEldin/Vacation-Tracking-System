# Vacation Tracking System (VTS)

## Vision:

A Vacation Tracking System (VTS) will provide individual employees with the
capability to manage their own vacation time, sick leave, and personal time off,
without having to be an expert in company policy or the local facility’s leave
policies.

## Functional Requirements:

■ Implements a flexible rules-based system for validating and verifying leave
time requests
■ Enables manager approval (optional)
■ Provides access to requests for the previous calendar year, and allows
requests to be made up to a year and a half in the future
■ Uses e-mail notification to request manager approval and notify employees
of request status changes
■ Uses existing hardware and middleware
■ Is implemented as an extension to the existing intranet portal system, and
uses the portal’s single-sign-on mechanisms for all authentication
■ Keeps activity logs for all transactions
■ Enables the HR and system administration personnel to override all actions
restricted by rules, with logging of those overrides
■ Allows managers to directly award personal leave time (with system-set
limits)
■ Provides a Web service interface for other internal systems to query any
given employee’s vacation request summary
■ Interfaces with the HR department legacy systems to retrieve required
employee information and changes

## Non Functional

 easy to use, intuitive, and intelligent

## Constraints 

 1. Authentication

Ensure that only authenticated users can access the system.

Use the portal’s single sign-on (SSO) mechanism.

Check employee credentials before showing vacation data or allowing request submission.

2. Role

Employee role: Can create and manage their own vacation requests.

Manager role: Can approve or disapprove subordinate requests.

HR/Admin role: Can override rules and make changes with logging.

3. Time Zone

Store all vacation dates in a consistent server time zone (e.g., UTC).

Convert dates to the employee’s local time zone in the UI.

Ensure comparisons (past/future limits) consider the user’s local time.

4. Request Date Limits

Show past 6 months of vacation requests.

Allow creating requests up to 18 months in the future.

Validate that selected dates fall within this range.

5. Manager Approval

If the vacation type requires approval, route the request to the manager.

Track request status: Pending, Approved, Rejected.

Only managers or HR/Admin can approve or reject.

6. Disapprove Explanation

Require managers to provide a reason when rejecting a request.

Store explanation in the database.

Include explanation in email notifications to the employee.

7. Vacation Hours or Days

Allow both full-day and partial-day requests.

Ensure employee cannot request more hours than available.

Validation: hours or days must be positive and within balance limits.

8. Data Validation

Validate all required fields: category, dates, hours/days, title, description.

Check that dates do not overlap with existing approved requests.

Highlight errors and prevent submission until corrected.

9. Email Notification

Send email to manager for approval requests.

Send email to employee on approval or rejection.

Include relevant details: dates, hours, category, manager comments.

## Pseudocode

employee opens VTS portal

IF Portal.authenticate(employee) == SUCCESS THEN
    vacationData = Database.getVacationHistory(employee, prevMonths=6, nextMonths=18)
    DISPLAY vacationData to employee
ELSE
    DISPLAY "Authentication failed"
    EXIT use case
ENDIF

category = employee.selectVacationCategory(vacationData)

dates, hours = employee.selectDatesAndHours(calendarView)

title, description = employee.enterDetails()

request = CreateRequest(employee, category, dates, hours, title, description)

validationResult = Validator.validate(request)
IF validationResult == FAILURE THEN
    DISPLAY validation errors
    employee can edit or cancel request
    RETURN to step 4
ENDIF

Database.saveRequest(request)

IF request.requiresApproval THEN
    Email.send(manager, "Approval request", request.details)
    request.status = PENDING_APPROVAL
ELSE
    request.status = APPROVED
ENDIF

IF request.status == PENDING_APPROVAL THEN
    managerResponse = Manager.reviewRequest(request)
    IF managerResponse == APPROVE THEN
        request.status = APPROVED
    ELSE
        request.status = REJECTED
        request.rejectionReason = managerResponse.reason
    ENDIF

    Email.send(employee, "Request " + request.status, request.details)
ENDIF

DISPLAY "Request process complete"
