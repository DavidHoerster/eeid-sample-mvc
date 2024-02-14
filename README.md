# EEID Sample MVC App

### dotnet CLI create
Use this command line to create the shell of this app from the dotnet CLI:

```
dotnet new mvc -au SingleOrg --aad-instance https://<tenant_name>.ciamlogin.com/ --domain <tenant_name>.ciamlogin.com --tenant-id <tenant_guid> --client-id <application_id aka client_id>
```