FROM mcr.microsoft.com/windows/servercore:ltsc2022

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop';"]

# Enable WinRM
RUN winrm quickconfig -q ; \
    winrm set winrm/config/service '@{AllowUnencrypted=\"true\"}' ; \
    winrm set winrm/config/service/Auth '@{Basic=\"true\"}' ; \
    net user ansible ansiblelab /add ; \
    net localgroup administrators ansible /add

CMD ["cmd.exe"]

