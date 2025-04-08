# SSH - GOOGLEAUTHENTICATOR
#ssh #2fa #mfa
Create container for the lab:
```bash
docker run -dit -p 2222:22 --cap-add SYS_ADMIN --security-opt apparmor=unconfined --name testsshdotp ubuntu
e23b140f6a9c19c9c264bd85f86b10b7d97bc0e0c7f3eb2e50ff90279eca3525
root@pvelab:~# docker exec -it testsshdotp bash
root@4a4defae0739:/# apt-get update
root@4a4defae0739:/# apt-get install openssh-server vim-tiny  libpam-google-authenticator rsyslog -y
root@4a4defae0739:/# service ssh start
 * Starting OpenBSD Secure Shell server sshd                                                                                             [ OK ]
root@4a4defae0739:/# rsyslogd &
```
- `--cap-add SYS_ADMIN`: Adds the `SYS_ADMIN` capability, which is required for rsyslogd.
- `--security-opt apparmor=unconfined`: Disables AppArmor.

Execution of rsyslogd for troubleshooting(tail -f /var/log/auth.log):
```bash
root@4a4defae0739:/# adduser testuser
info: Adding user `testuser' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `testuser' (1001) ...
info: Adding new user `testuser' (1001) with group `testuser (1001)' ...
info: Creating home directory `/home/testuser' ...
info: Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for testuser
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
info: Adding new user `testuser' to supplemental / extra groups `users' ...
info: Adding user `testuser' to group `users' ...
```

Create tokens on user home:
```bash
root@4a4defae0739:/var/log# su - testuser
testuser@4a4defae0739:~$ google-authenticator

Do you want authentication tokens to be time-based (y/n) y
Warning: pasting the following URL into your browser exposes the OTP secret to Google:
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/testuser@4a4defae0739%3Fsecret%3D6FEZLDXUPJP67F5EGR667K4ZDY%26issuer%3D4a4defae0739
Your new secret key is: 6FEZLDXUPJP67F5EGR667K4ZDY
Enter code from app (-1 to skip): 289871
Code confirmed
Your emergency scratch codes are:
  82527441
  25453192
  20636406
  87047537
  22814504

Do you want me to update your "/home/testuser/.google_authenticator" file? (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) n

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```

Add the following line to the top of the file to enable Google Authenticator for SSH **/etc/pam.d/sshd**
```bash
auth required pam_google_authenticator.so nullok  
auth required pam_permit.so
```
Add or modify the following line to allow PAM authentication **/etc/ssh/sshd_config** & restart service.
```bash
KbdInteractiveAuthentication yes
ChallengeResponseAuthentication yes 
```
