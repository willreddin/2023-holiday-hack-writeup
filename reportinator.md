# Reportinator
There's a Pentest report. The elves have used an AI tool to write it so we need to check to see if the AI has halucinated findings. We assume the findings are legit, but we need to mark any that don't make sense. 

## Approach
1. Go through the report, familiarise myself with the findings and see if any of the findings don't make sense
2. Use an AI detector tool like [AI detector]('https://contentdetector.ai/') to see if there are any findings that come back with high possibility of being hallucinations. 

## 1. Vulnerable Active Directory Certificate Service-Certificate Template Allows Group/User Privilege Escalation
Apparently the pen testers used [Certipy]('https://github.com/ly4k/Certipy') to fid vulnerable AD certs. The README on Certipy seems to suggest you can use the find -vulnerable command, so this seems legit.

TICK

##  2. SQL Injection Vulnerability in Java Application
[SQlmap]('https://sqlmap.org/') was used to identify a SQL injection vulnerability. The Attached screenshot shows an example use of the tool. Again this seems like a legitimate write up of the finidng and not a halucination. OWASP does have an [ESAPI]('https://owasp.org/www-project-enterprise-security-api/') library to help with this.

TICK

## 3. Remote Code Execution via Java Deserialization of Stored Database Objects
Uses [ysoserial]('https://github.com/frohoff/ysoserial') tool to expolit unsfae deserialisation in Java apps. 

The finding and the references it makes (like to [CWE-502]('https://cwe.mitre.org/data/definitions/502.html') and the [OWASP cheat sheet]('https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html')) are legit. 

Interstingly the screenshot is lifted from the github README on ysoserial, just with candycan.exe replacing calc.exe.

## 4. Azure Function Application-SSH Configuration Key Signing Vulnerable to Principal Manipulation


## 5. Azure Key Vault-Overly Permissive Access from Azure Virtual Machine Metadata Service/Managed Identity

## 6. Stored Cross-Site Scripting Vulnerabilities

## 7. Browsable Directory Structure

## 8. Deprecated Version of PHP Scripting Language

## 9. Internal IP Address Disclosure