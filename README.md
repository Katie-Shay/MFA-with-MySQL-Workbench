# MFA-with-MySQL-Workbench
Ubuntu 18.04 - Using MFA with Mysql Workbench

For MySQL Workbench TCP/IP over SSH with libpam-google-authenticator enabled - Ubuntu 18.04.  
  
Info
________________________________________________________________________________
Google Authenticator can be used with MySQL Workbench, but to achieve this it is necessary to change how SSH and Google Authenticator work together.   
  
When MFA using libpam-google-authenticator is enabled on Ubuntu18.04 user login over SSH  most likely follows this sequence: 
1. The user is prompted to enter their password
2. The user is prompted to enter their two-factor pin

Because Mysql Workbench only prompts once for SSH credentials the above sequence will not work, thus any attempt to login via Workbench will fail. This can be fixed by having the user enter their password and pin back to back on a single line like this:
  
[password][two-factor-pin]   
  
Setup
________________________________________________________________________________
Install Google Authenticator  
sudo apt-get install libpam-google-authenticator  
  
  
Create a custom PAM config file called “mfa-auth” using common-auth as a base  
sudo cp /etc/pam.d/common-auth /etc/pam.d/mfa-auth  
sudo nano /etc/pam.d/mfa-auth  
  
Add and/or modify mfa-auth to include these lines  
auth [success=1 default=ignore] pam_unix.so nullok_secure try_first_pass  
auth required pam_permit.so try_first_pass  
  
The try_first_pass option: “Before prompting the user for their password, the module first tries the previous stacked mofule’s password in case that satisfies the module as well”.
linux.die.net/man/8/pam_unix



Edit /etc/pam.d/sshd and add Google Authenticator  
sudo nano /etc/pam.d/sshd  

Add these lines to the top of the file  
auth required pam_google_authenticator.so nullok forward_pass  
@include mfa-auth  

Comment out line  
@include common-auth  
As We want to use mfa-auth to authenticate SSH connections now instead of common-auth.  
  
  
Edit /etc/ssh/sshd_config to ensure SSH is using is using two-factor authentication  
sudo nano /etc/ssh/sshd_config  
  
Set ChallengeResponseAuthentication to yes  
ChallengeResponseAuthentication yes  
  
  
  
Restart SSH  
sudo systemctl restart ssh  

  
Set up Google Authenticator pin for each user with your desired settings  
sudo su username  
google-authenticator 
