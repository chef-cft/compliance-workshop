## InSpec on Windows

Create a file called audit.rb
```bash
code audit.rb
```

Otherwise, to create a new InSpec Profile run the following:
```bash
## Create a New InSpec Profile
$ inspec init profile <your_profile_name>

## Change Directory into your new Profile
$ cd <your_profile_name>

## Write your Controls in the example.rb
$ code controls\example.rb
```


Now add the following to the audit.rb file

```bash
# Windows Versions - Check for Min of Win 2012
# Win2016 - NT 10.0 | Win 2012 R2 - NT 6.3 | Win 2012 - NT 6.2
#

control 'WINDOWS VERSION' do
  impact 0.8
  title 'This test checks for a minimum Windows version of 2012 - NT 6.2.0'

  describe os.family do
    it { should eq 'windows' }
  end

  describe os.name do
    it { should eq 'windows_server_2012_r2_standard_evaluation' }
  end

  describe os.release do
    it { should > '6.2' }
  end
end
```

To execute this using InSpec, run the following command

```bash
inspec exec audit.rb
```

Add the following example for looping through Windows KB and Hotfixes

```bash
## Looping example WannaCry Vulnerability Check
control 'WINDOWS HOTFIX - LOOP' do
  impact 0.8
  title 'This test checks that a numberof Windows Hotfixs are installed - Looping Example'

  hotfixes = %w{ KB4012598 KB4042895 KB4041693 KB4041691 KB4041690 KB4041689 KB4041681 KB4039396 KB4038803 KB4038801 KB4038799 KB4038797 KB4038792 KB4038783 KB4038782 KB4038781 KB4038777 KB4038774 KB4038220 KB4034681 KB4034670 KB4034668 KB4034665 KB4034664 KB4034663 KB4034661 KB4034660 KB4034659 KB4034658 KB4032695 KB4032693 KB4025344 KB4025341 KB4025340 KB4025339 KB4025338 KB4025336 KB4025335 KB4025334 KB4025332 KB4025331 KB4022724 KB4022723 KB4022722 KB4022721 KB4022720 KB4022719 KB4022718 KB4022717 KB4022168 KB4019474 KB4019473 KB4019472 KB4019265 KB4019264 KB4019263 KB4019218 KB4019217 KB4019216 KB4019215 KB4019214 KB4019213 KB4016637 KB4016636 KB4016635 KB4015554 KB4015553 KB4015552 KB4015551 KB4015550 KB4015549 KB4015221 KB4015219 KB4015217 KB4013429 KB4013198 KB4012606 KB4012220 KB4012219 KB4012218 KB4012217 KB4012216 KB4012215 KB4012214 KB4012213 KB4012212 }

  describe.one do
    hotfixes.each do |hotfix|
      describe windows_hotfix(hotfix) do
        it { should_not be_installed }
      end
    end
  end
end
```

To execute this using InSpec, run the following command

```bash
inspec exec audit.rb
```

Is a particular package installed ?

```bash
control 'PACKAGE INSTALLED _ TELNET' do
  impact 0.8
  title 'This test checks that a package is installed'

  describe package('telnetd') do
    it { should_not be_installed }
  end
end
```

Is a particular Service installed ?

```bash
## service example
control 'SERVICE INSTALLED' do
  impact 0.8
  title 'This test checks the service is installed'

  describe service('DHCP Client') do
    it { should be_installed }
    it { should be_running }
  end
end
```

To execute this using InSpec, run the following command

```bash
inspec exec audit.rb
```
Use the InSpec Port resource to test HTTP and HTTPS

```bash
# Test HTTP port 80, is not listening and no protocol TCP, ICMP, UDP
describe port(80) do
    it { should_not be_listening }
    its('protocols') { should_not cmp 'tcp6' }
    its('protocols') { should_not include('icmp') }
    its('protocols') { should_not include('tcp') }
    its('protocols') { should_not include('udp') }
    its('protocols') { should_not include('udp6') }
    its('addresses') { should_not include '0.0.0.0' }
end

# Test HTTPS port 443, listening with TCP and UDP
describe port(443) do
    it { should be_listening }
    its('protocols') { should_not cmp 'tcp6' }
    its('protocols') { should_not include('icmp') }
    its('protocols') { should include('tcp') }
    its('protocols') { should include('udp') }
    its('protocols') { should_not include('udp6') }
    its('addresses') { should include '0.0.0.0' }
end
```

To execute this using InSpec, run the following command

```bash
inspec exec audit.rb
```

Use the http InSpec audit resource to test an http endpoint.

```bash
describe http('https://automate.automate-demo.com/ping',
              auth: {user: 'user', pass: 'test'},
              params: {format: 'html'},
              method: 'POST',
              headers: {'Content-Type' => 'application/json'},
              data: '{"data":{"a":"1","b":"five"}}') do
  its('status') { should cmp 200 }
  its('body') { should cmp 'pong' }
  its('headers.Content-Type') { should cmp 'text/html' }
end
```

Compliance of the OS settings on the windows client
- Check and verify Group Policy Settings (GPO) with reference to CIS Windows 10 1703 benchmark is begin applied
- When new monthly windows security patch is applied to the current image, to check if the new patches is successfully applied. Where possible, show the status BEFORE and AFTER the patch for comparison and highlight any errors etc..


```bash
control "xccdf_org.cisecurity.benchmarks_rule_18.8.18.3_L1_Ensure_Configure_registry_policy_processing_Process_even_if_the_Group_Policy_objects_have_not_changed_is_set_to_Enabled_TRUE" do
  title "(L1) Ensure 'Configure registry policy processing: Process even if the Group Policy objects have not changed' is set to 'Enabled: TRUE'"
  desc  "The \"Process even if the Group Policy objects have not changed\" option updates and reapplies policies even if the policies have not changed."
  impact 1.0
  describe registry_key("HKEY_LOCAL_MACHINE\\Software\\Policies\\Microsoft\\Windows\\Group Policy\\{35378EAC-683F-11D2-A89A-00C04FBBCFA2}") do
    it { should_not have_property "NoGPOListChanges" }
  end
  describe registry_key("HKEY_LOCAL_MACHINE\\Software\\Policies\\Microsoft\\Windows\\Group Policy\\{35378EAC-683F-11D2-A89A-00C04FBBCFA2}") do
    its("NoGPOListChanges") { should_not cmp == 0 }
  end
end

control "xccdf_org.cisecurity.benchmarks_rule_18.8.18.4_L1_Ensure_Turn_off_background_refresh_of_Group_Policy_is_set_to_Disabled" do
  title "(L1) Ensure 'Turn off background refresh of Group Policy' is set to 'Disabled'"
  desc  "This policy setting prevents Group Policy from being updated while the computer is in use."
  impact 1.0
  describe registry_key("HKEY_LOCAL_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System") do
    it { should_not have_property "DisableBkGndGroupPolicy" }
  end
end
```

To execute this using InSpec, run the following command

```bash
inspec exec audit.rb
```