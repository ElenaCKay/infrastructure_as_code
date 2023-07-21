# Orchestration with Terraform

Not using AWS cloud formation as this could then only be used on AWS. It is cloud dependent. Terraform is cloud independant and so it can be used on any cloud, local or hybrid. 

## Task terraform

- What is it
- Why use it and the benefits
- Who is using it everyone
- Install terraform in your system 
- Run terraform --version (check it is there)
- ensure its installed correctly and env var added to path
- share the github repo with above steps


#### How to download terraform on Windows:

1. **Download Terraform**:
   - Go to the official Terraform website: [Terraform Downloads](https://www.terraform.io/downloads.html)
   - Download the Windows 64-bit version of Terraform (usually a zip archive).

2. **Extract the Archive**:
   - Once the download is complete, locate the downloaded zip archive.
   - Right-click on the zip file and select "Extract All..." to extract its contents to a folder of your choice. You can use the built-in Windows extraction tool or a third-party file archiver like 7-Zip.

3. **Add Terraform to System Path**:
   - To use Terraform from any directory on the command prompt, you need to add the folder containing the Terraform executable to your system's PATH environment variable.
   - Here's how you can do it:
     - Open the Start menu and search for "Environment Variables."
     - Select "Edit the system environment variables."
     - Click the "Environment Variables..." button.
     - In the System Variables section, scroll down and find the "Path" variable. Click "Edit."
     - Click "New," and then add the path to the folder where you extracted the Terraform executable. For example, it might look like: `C:\path\to\terraform`.

4. **Verify Installation**:
   - Open a new Command Prompt (or PowerShell) window.
   - Type `terraform --version` and press Enter.
   - If Terraform is correctly installed and added to the system PATH, it will display the version information.
   - 
![Alt text](imgs/terraform-downloaded.png)

