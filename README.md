# AWS Emulator on Docker (Windows)
 
Notes for running an AWS emulator in Docker on Windows and using it with the AWS CLI.
 
## Prerequisites
 
- Docker Desktop
- PowerShell
- A `Dockerfile` in the project folder (`D:\PROJECTS\AWS Emulator`)
## 1. Install AWS CLI
 
Run in PowerShell:
 
```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2-User.msi
```
 
The user installer puts `aws.exe` here:
 
```
D:\Users\<your-username>\AppData\Local\Programs\Amazon\AWSCLIV2\aws.exe
```
 
Add that folder to your user `Path` so new terminals can find `aws`:
 
```powershell
$dir = "D:\Users\<your-username>\AppData\Local\Programs\Amazon\AWSCLIV2"
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
if (($userPath -split ';') -notcontains $dir) {
  if ($userPath) { $userPath = "$userPath;$dir" } else { $userPath = $dir }
  [Environment]::SetEnvironmentVariable("Path", $userPath, "User")
}
$env:Path = "$env:Path;$dir"
aws --version
```
 
> **Note:** Restart any terminal that was already open. It won't see the updated `Path` until it is reopened.
 
## 2. Build the image
 
From the project folder (where the `Dockerfile` is):
 
```powershell
cd "D:\PROJECTS\AWS Emulator"
docker build -t aws-emulator .
```
 
## 3. Create the data folder
 
Create this folder:
 
```
D:\PROJECTS\AWS Emulator\.aws\emulator
```
 
## 4. Run the emulator
 
```powershell
$port = 9297
docker run --rm --memory=2g --memory-swap=2g -p "127.0.0.1:${port}:${port}" `
  -v "D:\PROJECTS\AWS Emulator\.aws\emulator:/home/emulator/.aws/emulator" `
  aws-emulator -p $port
```
 
## 5. Point the AWS CLI to the emulator
 
Open a **new** PowerShell window and run:
 
```powershell
$env:AWS_ENDPOINT_URL = "https://127.0.0.1:9297"
$env:AWS_SHARED_CREDENTIALS_FILE = "D:\PROJECTS\AWS Emulator\.aws\emulator\9297\emulator_credentials"
$env:AWS_CONFIG_FILE = "D:\PROJECTS\AWS Emulator\.aws\emulator\9297\config"
$env:AWS_CA_BUNDLE = "D:\PROJECTS\AWS Emulator\.aws\emulator\9297\ca.pem"
$env:AWS_EC2_METADATA_DISABLED = "true"
$env:AWS_DISABLE_HOST_PREFIX_INJECTION = "true"
$env:AWS_REGION = "us-east-1"
$env:AWS_DEFAULT_REGION = "us-east-1"
```
 
Clear any real AWS credentials so they don't interfere:
 
```powershell
Remove-Item Env:AWS_ACCESS_KEY_ID, Env:AWS_SECRET_ACCESS_KEY, Env:AWS_SESSION_TOKEN, Env:AWS_PROFILE -ErrorAction SilentlyContinue
```
 
## 6. Verify
 
Check emulator health (CMD):
 
```cmd
curl -k https://127.0.0.1:9297/__admin/health
```
 
Check the current identity:
 
```powershell
aws sts get-caller-identity
```
 
## 7. Try it out with S3
 
```powershell
aws s3 mb s3://local-s3
echo "hello asad" > sample.txt
aws s3 cp sample.txt s3://local-s3
aws s3 ls s3://local-s3
```
 
