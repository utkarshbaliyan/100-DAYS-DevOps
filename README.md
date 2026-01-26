# 100-DAYS-DevOps
I am starting Devops from absolute basics and will go advance in next 100 days 


Why DevOps?
1. The Traditional Workflow ("The Silo Era")
Before DevOps, software development followed a rigid, linear process often referred to as the "Throw it over the wall" approach. Teams worked in isolation (silos).

The Process:
Developers: Wrote code, "packaged" it (often just a simple .zip file), and handed it off.
Operations Team: Received the package and attempted to deploy it.
System Admins: Configured the OS and servers.
Network Admins: Opened ports and configured firewalls.
Database Admins (DBAs): Updated schemas and database tables.
QA / Testers: Finally tested the application after deployment.

2. The Problems with the Traditional Approach

The workflow described above created several critical issues that hurt the business:
🐢 Slow Time to Market: Because the code had to physically pass through so many different teams—each with their own queues and priorities—deployments could take weeks or months.
🚧 The "Works on My Machine" Syndrome: Developers packaged code in a zip file based on their local environment. When Ops tried to run it on a server, it often failed because the environments (OS versions, libraries, dependencies) were different.
👥 Resource Intensive: As you noted, this required a massive amount of people and manual effort. Every deployment required the synchronized time of Devs, Ops, Admins, and QA.
👉 The Blame Game: When software broke, Developers blamed Operations for bad servers, and Operations blamed Developers for bad code. There was no shared ownership.

3. The DevOps Solution

DevOps was introduced to break down these silos. It merges Development and Operations into a single, continuous loop.

Feature	             Traditional / Pre-DevOps	          DevOps Approach
Handoff	             Manual (Zip files passed to Ops)	  Automated (CI/CD Pipelines)
Responsibility	     Siloed (Devs code, Ops deploy)	    Shared (You build it, you run it)
Speed                Monthly or Quarterly releases	    Daily or Weekly releases
Infrastructure	     Manual setup by Admins	            Infrastructure as Code (IaC)




