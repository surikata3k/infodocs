# Remote PowerShell

To connect remotely with **PowerShell** to a Windows Server 2025, the standard method is **WinRM** (*Windows Remote Management*) with the `Enter-PSSession` cmdlet.

## 1. Prepare the server

In Windows Server 2025, remote management is usually enabled by default. To check or enable it, run this in PowerShell as **Administrator** on the server:

```powershell
Enable-PSRemoting -Force
```

## 2. Connect from the client

=== "With a domain (Active Directory)"

    ```powershell
    Enter-PSSession -ComputerName "NomDelServidor" -Credential (Get-Credential)
    ```

    !!! note
        Enter the credentials of a user with administrator permissions on the server.

=== "Without a domain (workgroup)"

    Without Active Directory, add the server to the *TrustedHosts* list of the client.

    1.  On the client, open PowerShell as **Administrator** and run:

        ```powershell
        Set-Item WSMan:\localhost\Client\TrustedHosts -Value "IP_DEL_SERVIDOR" -Concatenate -Force
        ```

        *(Optional)* Use `"*"` instead of the IP to trust any computer on the local network.

    2.  Connect using the server IP:

        ```powershell
        Enter-PSSession -ComputerName "IP_DEL_SERVIDOR" -Credential "IP_DEL_SERVIDOR\Administrador"
        ```
