# Security Practices

This repository aims to demonstrate good security practices, as it is common to unintentionally maintain sensitive data in files committed to GitHub. Accidental inclusion of sensitive information can happen, and this repository will showcase practices I have found helpful in preventing such mistakes.

## Overview

It is important to note that GitHub works as a Git history manager, meaning you can track all modifications based on specific commits. If any commit contains sensitive data, anyone with access to the repository can potentially find that data, which can be harmful to the owner or the organization.

## Good Practices

1. **Keep Sensitive Data in a `.env` File**  
   Store sensitive data in a `.env` file and access it as environment variables. After creating the `.env` file, create (if it doesn't exist) a `.gitignore` file with the path to the `.env` file. This ensures that the `.env` file will not appear as a modified file ready to commit in your repository.

2. **Use Git Hooks to Prevent Sensitive Data from Being Committed**  
   To avoid accidentally adding sensitive data to any files and committing them, you can use Git Hooks by creating a `bash` script to check if any lines in your committed files contain suspicious sensitive data. You can create a `pre-commit` file in the following path: `.git/hooks/`, and execute the command `chmod +x .git/hooks/pre-commit` to make it executable.

   ### Example: Git Pre-Commit Hook Script

   This script checks for sensitive data in staged files before a commit is made. If sensitive data is detected, the commit will be rejected.

   ```bash
   #!/bin/bash

   # Define patterns for sensitive data (add as needed)
   regexes=(
       'AKIA[0-9A-Z]{16}'                                    # AWS Access Key
       'AWS_SECRET_ACCESS_KEY=[0-9a-zA-Z+/=]{40}'          # AWS Secret Access Key
       'AIza[0-9A-Za-z_-]{35}'                               # Google API Key
       'ghp_[0-9A-Za-z]{36}'                                 # GitHub Token
       '[a-zA-Z0-9]{24}\.[a-zA-Z0-9]{6}\.[a-zA-Z0-9_-]{27}' # JWT Token
       'DefaultEndpointsProtocol=https;AccountName=[a-zA-Z0-9]+;AccountKey=[a-zA-Z0-9+/=]+;EndpointSuffix=core.windows.net' # Azure Blob Storage
       'AccountName=[a-zA-Z0-9]+;AccountKey=[a-zA-Z0-9+/=]+;Endpoint=https://[a-zA-Z0-9]+.api.azureml.ms;' # Azure ML Workspace
       'AKIA[0-9A-Z]{16}|[0-9a-zA-Z+/=]{40}'                 # AWS Secret Access Key
       '"client_id": "[0-9a-f]{32}",\s*"client_secret": "[0-9a-zA-Z]{40}",\s*"tenant": "[0-9a-f]{32}"' # Azure Service Principal
       'AIza[0-9A-Za-z-_]{35}'                               # Firebase API Key
       'postgres://[a-zA-Z0-9_.+-]+:[^@]+@[^:]+:[0-9]+/[a-zA-Z0-9_.+-]+' # PostgreSQL
       'mysql://[a-zA-Z0-9_.+-]+:[^@]+@[^:]+:[0-9]+/[a-zA-Z0-9_.+-]+'  # MySQL
       'redis://:[0-9a-f]{40}@([a-zA-Z0-9.-]+:[0-9]+|localhost:[0-9]+)' # Redis
       'amqp://[a-zA-Z0-9_.+-]+:[^@]+@[^:]+:[0-9]+'         # RabbitMQ
       'mongodb://[a-zA-Z0-9_.+-]+:[^@]+@[^:]+:[0-9]+/[a-zA-Z0-9_.+-]+' # MongoDB
   )

   # Scan for sensitive data in staged files
   for file in $(git diff --cached --name-only); do
       for regex in "${regexes[@]}"; do
           if grep -qE "$regex" "$file"; then
               echo "🚨 Sensitive data detected in $file. Commit rejected."
               exit 1
           fi
       done
   done

   echo "✅ No sensitive data found. Proceeding with commit."
   exit 0

The code above is a general example of how to check for common sensitive data in files and automatically abort your commit if such data is detected.

3. **Removing Sensitive Data from Git History**
If you accidentally push a file containing sensitive data, remain calm. It is possible to remove the file from Git history. The following link explains how to remove the file from the entire Git history.

4. **Regularly Review and Audit Your Repositories**
Regularly review your repositories to identify and remove any sensitive data that may have been unintentionally committed. Consider conducting periodic audits to ensure compliance with security best practices.

5. **Use Third-Party Tools for Scanning**
Leverage third-party tools, such as GitGuardian or TruffleHog, to automatically scan your repositories for sensitive information. These tools can provide an additional layer of security and alert you to potential leaks.

By following these practices, you can significantly reduce the risk of exposing sensitive data in your repositories.