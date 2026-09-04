# RefreshUser Performance Test

Apache JMeter performance test for refreshing selected users in the
HMCTS Manage Organisation performance-test environment.

The test logs in, opens the Users area, filters the organisation user
list using email addresses from a CSV file, retrieves each matching
user's details, builds the required refresh payload, submits the update,
and records a `SUCCESS` or `SORRY` result.

> **Security:** The source test plan contains a hardcoded test
> credential. Do not commit real credentials to source control. Replace
> credentials with secure runtime properties, environment variables, or
> another approved secret-management mechanism before sharing or running
> the test outside its intended test environment.

## Requirements

-   Apache JMeter **5.6.3** or compatible version.
-   Network access to the configured performance-test endpoints.
-   A CSV file named `RefreshUser_emails.csv`.
-   A valid test account with permission to access and update users in
    the target environment.
-   Groovy support provided by JMeter's JSR223 components.

## Files

The test plan expects the following input file in JMeter's working/base
directory:

``` text
RefreshUser_emails.csv
```

The CSV should contain an `email` column. The first row is ignored as a
header.

Example:

``` csv
email
user1@example.com
user2@example.com
```

## Configuration

The test plan defines these user variables:

  Variable         Purpose
  ---------------- ---------------------------------------
  `BASE_URL_1`     Manage Organisation host
  `BASE_URL_2`     IDAM web/public host
  `username`       Login email/username
  `password`       Login password
  `Thinktime`      Post-login think time in milliseconds
  `emailCsvPath`   Path to `RefreshUser_emails.csv`
  `outputStamp`    Timestamp used in output filenames

The configured performance-test hosts are:

``` text
https://manage-org.perftest.platform.hmcts.net
https://idam-web-public.perftest.platform.hmcts.net
```

For secure execution, override credentials at runtime rather than
storing them in the `.jmx` file.

## Test Flow

The test executes the following high-level flow:

1.  **Login**
    -   `GET /auth/login`
    -   Extracts the CSRF token.
    -   `POST /enter-email`
    -   `GET /enter-password`
    -   `POST /enter-password`
2.  **Open Users**
    -   `GET /users`
    -   Retrieves the XSRF token from the response/cookie.
    -   Verifies the Users page returns HTTP `200`.
3.  **Retrieve supporting data**
    -   `GET /external/configuration-ui/`
    -   `GET /auth/isAuthenticated`
    -   `GET /api/user/details`
    -   `GET /api/healthCheck?path=%2Fusers`
    -   `GET /api/allUserListWithoutRoles`
    -   `POST /api/retrieve-access-types`
4.  **Match CSV users**
    -   Reads email addresses from `RefreshUser_emails.csv`.
    -   Matches them case-insensitively against the organisation user
        list.
    -   Preserves the order from the CSV.
    -   Stores matched user IDs and email addresses for iteration.
    -   Emails not found in the user list are recorded as `SORRY`.
5.  **Refresh each matched user**
    -   `GET /api/user-details?userId=<userId>`
    -   Reads the user's IDAM status, roles, identity and access types.
    -   Detects whether the `pui-case-manager` role is already present.
    -   Builds the update payload.
    -   `PUT /api/ogd-flow/update/<userId>`
6.  **Record the outcome**
    -   The update response is inspected for HTTP status and IDAM
        response information.
    -   The result is classified as `SUCCESS` or `SORRY`.
    -   A CSV row is written for each processed user.

## Role Handling

When `pui-case-manager` is not already present, the test adds the
case-management roles that are missing from the user's current role
list.

The role set considered by the test includes:

-   `pui-case-manager`
-   `caseworker`
-   `caseworker-divorce`
-   `caseworker-divorce-solicitor`
-   `caseworker-divorce-financialremedy`
-   `caseworker-divorce-financialremedy-solicitor`
-   `caseworker-probate`
-   `caseworker-ia`
-   `caseworker-probate-solicitor`
-   `caseworker-publiclaw`
-   `caseworker-ia-legalrep-solicitor`
-   `caseworker-publiclaw-solicitor`
-   `caseworker-civil`
-   `caseworker-civil-solicitor`
-   `caseworker-employment`
-   `caseworker-employment-legalrep-solicitor`
-   `caseworker-privatelaw`
-   `caseworker-privatelaw-solicitor`
-   `caseworker-pcs`
-   `caseworker-pcs-solicitor`

If case management is already checked, the test does not add those
roles.

## Access Types

The refresh payload ensures the following access types are present:

  --------------------------------------------------------------------------------------------
  Jurisdiction      Organisation profile  Access type                        Enabled
  ----------------- --------------------- ---------------------------------- -----------------
  `BEFTA_MASTER`    `SOLICITOR_PROFILE`   `BEFTA_SOLICITOR_1`                `true`

  `PCS`             `SOLICITOR_PROFILE`   `solicitor-org-claimant-access`    `true`

  `PCS`             `SOLICITOR_PROFILE`   `solicitor-org-defendant-access`   `true`

  `PCS`             `SOLICITOR_PROFILE`   `duty-advisor-access`              `false`
  --------------------------------------------------------------------------------------------

Existing access types are retained and merged with these required
values.

## Running the Test

### GUI mode

Open the test plan in JMeter:

``` bash
jmeter -t RefreshUser_PerfTest.jmx
```

Check the User Defined Variables and ensure:

-   the target hosts are correct;
-   the login credentials are supplied securely;
-   `RefreshUser_emails.csv` exists;
-   the CSV contains the expected email addresses.

Run the test from the JMeter GUI and review the **Aggregate Report** and
**View Results Tree**.

### Non-GUI mode

For a repeatable performance run, use JMeter's non-GUI mode:

``` bash
jmeter -n -t RefreshUser_PerfTest.jmx
```

If overriding variables at runtime:

``` bash
jmeter -n \
  -t RefreshUser_PerfTest.jmx \
  -Jusername="<test-user>" \
  -Jpassword="<test-password>"
```

Use the credential injection approach approved for your environment
rather than placing secrets in shell history or source control.

## Output

The test writes timestamped CSV files to JMeter's base/working
directory.

### All results

``` text
output_RefreshUser_<timestamp>.csv
```

### Successful results

``` text
output_RefreshUser_success_<timestamp>.csv
```

### Failed results

``` text
output_RefreshUser_sorry_<timestamp>.csv
```

Each output file uses the following columns:

``` text
email,userId,idamStatus,cmaState,httpCode,result,message
```

Where:

  Column         Description
  -------------- ------------------------------------------------
  `email`        User email processed
  `userId`       User identifier
  `idamStatus`   IDAM status returned by user details
  `cmaState`     Whether `pui-case-manager` was already present
  `httpCode`     HTTP response code from the relevant operation
  `result`       `SUCCESS` or `SORRY`
  `message`      Human-readable outcome/details

Users that are present in the CSV but cannot be found in the
organisation user list are written to the `SORRY` output with an
explanatory message.

## Result Criteria

A refresh is recorded as `SUCCESS` when:

-   the update returns HTTP `200`;
-   the response does not contain a detected "sorry" condition; and
-   the IDAM response objects do not report a non-`200` IDAM status or
    recognised failure message.

Otherwise, the result is recorded as `SORRY`.

For users whose IDAM status is `PENDING`, the output message records
that the user details loaded and the refresh was submitted.

## CSRF/XSRF Handling

The test dynamically extracts and reuses security tokens:

-   CSRF tokens are extracted from IDAM HTML pages.
-   The `XSRF-TOKEN` cookie is extracted from the organisation
    application.
-   The XSRF token is supplied as the `x-xsrf-token` header for JSON API
    requests.

This avoids relying on static token values.

## Test Structure

The main JMeter transaction groups are:

``` text
RefreshUser_PerfTest
└── RefreshUser_ThreadGroup
    ├── RefreshUser_01_Login
    ├── Thinktime_After_Login
    ├── RefreshUser_02_Open_Users
    ├── RefreshUser_03_ForEach_Filter_Email
    │   └── RefreshUser_04_Refresh_Selected_User
    │       ├── GET /api/user-details
    │       └── RefreshUser_05_If_Details_OK_Then_Change_Submit
    │           └── PUT /api/ogd-flow/update/<userId>
    └── Thinktime_After_Refresh
```

The configured thread group uses **1 thread**, a **1 second ramp-up**,
and **1 loop**. The CSV is intentionally not used as a standard JMeter
thread CSV dataset; the Groovy processor loads and matches the email
list so that login is not repeated once per CSV row.

## Assertions

The test contains response assertions for key steps, including:

-   Login email page
-   Password page
-   Users page
-   Configuration UI
-   User authentication/details
-   User list
-   Access types
-   Refresh/update response

Most API assertions expect HTTP `200`.

## Troubleshooting

### No users are processed

Check that:

1.  `RefreshUser_emails.csv` is in JMeter's base directory.
2.  The CSV contains an `email` header and valid addresses.
3.  The addresses match users returned by
    `/api/allUserListWithoutRoles`.
4.  The Users API returned HTTP `200`.

The JMeter log reports the number of matched emails.

### `SORRY` for user details

Check:

-   the user's ID is valid;
-   `/api/user-details?userId=<userId>` returns JSON;
-   the response is HTTP `200`;
-   the response does not contain an unexpected HTML/error page.

### `SORRY` after the update

Inspect the generated `output_RefreshUser_sorry_<timestamp>.csv` and the
JMeter logs. The test captures relevant IDAM status/message information
when available.

### XSRF errors

Confirm that the login/session has been established and that the
`XSRF-TOKEN` cookie is available. The test attempts to copy the token
from the Cookie Manager before protected API requests.

## Notes

-   The test is configured specifically for the performance-test
    environment shown in the source test plan.
-   The test mutates user data through
    `PUT /api/ogd-flow/update/<userId>`, so it should only be run
    against an approved test environment and test accounts.
-   The source test plan includes a 3-second think time after login and
    a 1-second delay after each refresh.
-   JMeter result listeners include **View Results Tree** and
    **Aggregate Report**.
