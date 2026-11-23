EXUI Solicitor User Journey
This repository contains JMeter scripts for the EXUI Solicitor User Journey, covering the process of viewing and creating possession claim cases via the XUI interface.
User Journey 

1.	CaseList
Retrieves and displays existing possession claim cases.
2.	Create Case
Creates a new possession claim case using the required input details.
Repository Structure

```text
.
├── scripts/
│   └── UserJourney/
│       └── XUI/
│           └── Solicitor-CaseView/
│               ├── CaseListView.jmx
│               └── Solicitor-CreateCase.jmx
├── Files/
│   └── (PDF, JPG, DOC test files)
├── config/
│   └── (environment and configuration files)
├── data/
│   └── (CSV and test data files)
├── lib/
│   └── (custom JMeter libraries and dependencies)
├── results/
│   └── (JMeter execution results and reports)
├── README.md
└── (other supporting files)
```



Prerequisites
```text
•	Java 8 or later
•	Apache JMeter 5.5
```

•	Git Setup Instructions
```
    1.	Clone the repository:
            git clone <repository-url>
    2.	Open Apache JMeter.
    
    3.	Load the JMeter test plans:
    
        1.	SN1 – CaseListView
            scripts/UserJourney/XUI/Solicitor-CaseView/CaseListView.jmx
        2.	SN2 – CreateCase
            scripts/UserJourney/XUI/Solicitor-CaseView/Solicitor-CreateCase.jmx
```

Test Data
```text
The Files folder contains sample documents used during execution:
•	5 MB PDF file
•	1 MB JPG file
•	1 MB DOC file

File paths are parameterized using the FILEPATH variable in JMeter’s User Defined Variables, ensuring portability across different machines.
```
Multiple Users Execution
```text
To execute the scripts with multiple users:

•	Add a CSV file containing user details.
•	Configure the CSV Data Set Config element in JMeter to reference the CSV file.

The userdetails CSV files in this repository contain only the header row.
Add test user rows locally before running, and do not commit them.
```
Environment Configuration
```text
Backend service host names are not stored in the scripts. They are read from
JMeter properties (s2sHost, pcsApiHost, ccdDataStoreHost, dmStoreHost).

1.	Copy config/env.properties.example to config/env.properties (git-ignored).
2.	Fill in the host names for the target environment.
3.	Pass the file to JMeter with -q (see below).
```
Running the Tests
```text
GUI Mode
Run the scripts directly from the JMeter GUI for debugging and validation:

**jmeter -q config/env.properties**

Non-GUI Mode (Recommended)
Execute the following command from the terminal or command prompt:

**jmeter -q config/env.properties -n -t <test_plan>.jmx -l results.jtl**

Command Line Parameters

•	-q – Additional JMeter properties file (environment host names)
•	-n – Runs JMeter in non-GUI mode (faster and more reliable)
•	-t – Path to the JMeter test plan (.jmx) file
•	-l – Output results file (.jtl)

Results files (.jtl, Results folders) are git-ignored and should not be committed.
```
Notes
```text
•	Ensure all variables and file paths are correctly configured before execution.
•	Non-GUI mode is recommended for CI/CD pipelines and performance execution
```
