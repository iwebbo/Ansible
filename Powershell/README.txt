Run as administrator of PowerShell
set-executionpolicy unrestricted

If bad generation of certificate need to be delete (before to re run script - setup_certificate_winrm.ps1) 
$CertThumbprint = "A8EE82A4C05AB8B835B688F25D3D79932E4E652C"
Get-ChildItem -Path "Cert:\LocalMachine\My\" | Where-Object { $_.Thumbprint -eq $CertThumbprint } | Remove-Item -Force
