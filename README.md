EXUI Solicitor User Journey
This repository contains JMeter scripts for the EXUI Solicitor User Journey, covering the process of viewing and creating possession claim cases via the XUI interface.
User Journey
1.	CaseList
Retrieves and displays existing possession claim cases.
2.	Create Case
Creates a new possession claim case using the required input details.
Repository Structure

.
├── scripts/
│ └── UserJourney/
│ └── XUI/
│ └── Solicitor-CaseView/
│ ├── CaseListView.jmx
│ └── Solicitor-CreateCase.jmx
├── Files/
│ └── (PDF, JPG, DOC test files)
├── config/
│ └── (environment and configuration files)
├── data/
│ └── (CSV and test data files)
├── lib/
│ └── (custom JMeter libraries and dependencies)
├── results/
│ └── (JMeter execution results and reports)
├── README.md
└── (other supporting files)

Prerequisites
•	Java 8 or later
•	Apache JMeter 5.5
•	Git
Setup Instructions
1.	Clone the repository:
git clone <repository-url>
2.	Open Apache JMeter.
3.	Load the JMeter test plans:
1.	SN1 – CaseListView
scripts/UserJourney/XUI/Solicitor-CaseView/CaseListView.jmx
2.	SN2 – CreateCase
scripts/UserJourney/XUI/Solicitor-CaseView/Solicitor-CreateCase.jmx
Test Data
The Files folder contains sample documents used during execution:
•	5 MB PDF file
•	1 MB JPG file
•	1 MB DOC file
File paths are parameterized using the FILEPATH variable in JMeter’s User Defined Variables, ensuring portability across different machines.
Multiple Users Execution
To execute the scripts with multiple users:
•	Add a CSV file containing user details.
•	Configure the CSV Data Set Config element in JMeter to reference the CSV file.
Running the Tests
GUI Mode
Run the scripts directly from the JMeter GUI for debugging and validation.
Non-GUI Mode (Recommended)
Execute the following command from the terminal or command prompt:
jmeter -n -t <test_plan>.jmx -l results.jtl
Command Line Parameters
•	-n – Runs JMeter in non-GUI mode (faster and more reliable)
•	-t – Path to the JMeter test plan (.jmx) file
•	-l – Output results file (.jtl)
Notes
•	Ensure all variables and file paths are correctly configured before execution.
•	Non-GUI mode is recommended for CI/CD pipelines and performance execution
