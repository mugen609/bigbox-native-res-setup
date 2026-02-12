# Gamepad-Only Workflow: SSH Remote Access

0. **Note the Emulation PC’s IP Address**  
   - Open Terminal/CMD and run:  
     ```cmd
     ipconfig
     ```  
   - Write down the IPv4 address of the active adapter.

1. **Still on the Emulation PC → Enable OpenSSH Server**

   - Open PowerShell as administrator and run:
     ```powershell
     Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
     ```
   - Verify installation:
     ```powershell
     Get-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
     ```

2. **Configure SSH Service**

   - Run `services.msc`.  
   - Find **OpenSSH SSH Server (sshd)**, set Startup type to **Automatic**, and click **Start**.  
   - Reboot.

3. **On Your Main PC → Generate a Key with WinSCP**

   - Open WinSCP → create/edit a session → click **Advanced…**  
   - Go to: SSH → Authentication.  
   - Click **Tools → Generate New Key Pair with PuTTYgen**.  
   - In PuTTYgen, choose **EdDSA (Ed25519)**, generate the key, and save the private key (`.ppk` file) somewhere safe.  
   - Back in WinSCP, click **Display Public Key** and copy the public key text (OpenSSH `authorized_keys` format).

4. **On the Emulation PC → Install the Public Key (Admin)**

   - Create a blank file named `administrators_authorized_keys` in the folder  
     `C:\ProgramData\ssh\`
 
   - Paste the public key as a single line into the file (one key per line) and save.

5. **Fix Permissions**

   - Open PowerShell as Administrator and run:
     ```powershell
     icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
     ```

6. **Configure SSH to Use Keys**

   - Edit `C:\ProgramData\ssh\sshd_config` and ensure:
     ```text
     PubkeyAuthentication yes
     PasswordAuthentication no
     ```
   - Restart the **OpenSSH SSH Server (sshd)** service in `services.msc` or reboot.

7. **On Your Main PC → Set up WinSCP Connection**

   - File protocol: **SFTP**  
   - Hostname: emulation PC IP  
   - Port: 22  
   - Username: emulation PC’s Windows admin user  
   - In **Advanced → SSH → Authentication**, set *Private key file* to the `.ppk` you saved.

 > SSH Notes
 > - *No screen passwords — Windows stays auto-login*
 > - *USB fallback available but less convenient*
 > - *CRU bonus: Export .bin → WinSCP → main PC backup*