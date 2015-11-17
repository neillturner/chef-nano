# chef-nano

HACK1
HACK2
HACK3
## Description

chef-repo for demostrating test-kitchen and chef-solo with nano server

it using a virtual box build by mwrock

see http://www.hurryupandwait.io/blog/a-packer-template-for-windows-nano-server-weighing-300mb


## Installation

On your windows machine

1. install latest virtual box

2. install latest vagrant via msi

3. install ruby windows 2.x

4. gem install test-kitchen

5. gem install kitchen-vagrant

6. gem install winrm-transport

7. gem install winrm -v 1.3.5.dev

8. gem install vagrant-winrm

9. vagrant plugin install vagrant-winrm

## Server Creation

To create the server in a command window change directory to this folder and run:

kitchen create base-WindowsNano

## Connect to server via powershell

run:
```
# Enable powershell remoting if it is not already enabled
Enable-PSRemoting -Force

# You may change "*" to the name or IP of the machine you want to connect to
Set-Item "wsman:\localhost\client\trustedhosts" -Value "*" -Force

# the password is vagrant
$creds = Get-Credential vagrant

# this assumes you are using NAT'd network which is the Virtualbox default
# Use the computername or IP of the machine mand skip the port arg
# if you are using Hyper-V or another non NAT network
Enter-PSSession -Computername localhost -Port 55985 -Credential $creds
```


## Issues

* WinRM connectivity from Vagrant for Nano Server
The WinRM service in Nano currently only accepts requests set to the UTF-8 code page 65001. However, the WinRM Gem used by Vagrant sets the codepage to 437.
This prevents vagrant from establishing WinRM connections with the nano box.
to workaround this:
```
copy winrm-1.3.5.dev from your gem folder to C:\HashiCorp\Vagrant\embedded\gems\gems.

remove the old C:\HashiCorp\Vagrant\embedded\gems\gems\winrm-1.3.3

rename new C:\HashiCorp\Vagrant\embedded\gems\gems\winrm-1.3.5.dev as
C:\HashiCorp\Vagrant\embedded\gems\gems\winrm-1.3.3
```

* Getting error 'Unquoted fields do not allow \r or \n (line 2)'
To workaround this problem:
change winrm-transport-1.0.2\lib\winrm\transport\file_transporter.rb
```
array =  CSV.parse(output.stdout, :headers => true).map(&:to_hash)
```
to
```
response_text = output.stdout
response_text.delete!("\n")
response_text.delete!("\r")
array =  CSV.parse(response_text, :headers => true).map(&:to_hash).
```

* CommandNotFoundException after running Chef Solo
```
 $env:systemdrive\opscode\chef\bin\chef-solo.bat --config $env:TEMP\kitchen\solo.rb --log_level debug
 --force-formatter --no-color --json-attributes $env:TEMP\kitchen\dna.json)

  FullyQualifiedId: CommandNotFoundException
     ExceptionName: System.Management.Automation.CommandNotFoundException
       InnerExName: null
     Error Message: The term 'CgAkAGUAbgB2ADoAUABBAFQASAAgAD0AIABbAFMAeQBzAHQAZQ
       StackTrace: Check .\App_path\Stacktrace.txt for more information
```
 This is currently outstanding. Currently this early version of nano does not support encrypted communication which what the kitchen winrm works with. Laer version of nano will support encryption and then test-kitchen will work completely with Windows Nano.
