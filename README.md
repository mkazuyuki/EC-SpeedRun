# EC-SpeedRun

## Configuring VM on Hyper-V

```ps1
Param(
	[parameter(mandatory=$false)][String]$InstallMediaPath = "C:\Users\Administrator\Downloads\en-us_windows_server_2025_updated_april_2025_x64_dvd_ea86301d.iso",
	[parameter(mandatory=$false)][int]$CPUCores   = 4,
	[parameter(mandatory=$false)][long]$MEMORY    = 8GB,
	[parameter(mandatory=$false)][long]$DISK1size = 50GB,
	[parameter(mandatory=$false)][long]$DISK2SIZE = 50GB,
	[parameter(mandatory=$false)][String]$vSwitch = "Virtual Switch Internal",

	[parameter(mandatory=$false)][String]$VM1NAME = "ws25-1",
	[parameter(mandatory=$false)][String]$VM1DISK1 = "C:\HyperV\$VM1name.vhdx",
	[parameter(mandatory=$false)][String]$VM1DISK2 = "C:\HyperV\$VM1name-2.vhdx",

	[parameter(mandatory=$false)][String]$VM2NAME = "ws25-2",
	[parameter(mandatory=$false)][String]$VM2DISK1 = "C:\HyperV\$VM2name.vhdx",
	[parameter(mandatory=$false)][String]$VM2DISK2 = "C:\HyperV\$VM2name-2.vhdx"
)

function makeVM{
	New-VM -Name $VMNAME -MemoryStartupBytes $MEMORY -NewVHDPath $VMDISK1 -NewVHDSizeBytes $DISK1size -SwitchName $vSwitch -Generation 2
	Add-VMNetworkAdapter -VMName $VMNAME -SwitchName $vSwitch -Name "NIC2"
	New-VHD -Path $VMDISK2 -SizeBytes $DISK2SIZE -Dynamic
	Add-VMHardDiskDrive -VMName $VMNAME -Path $VMDISK2
	Set-VMProcessor $VMNAME -Count $CPUCores
	Set-VMMemory -VMName $VMNAME -DynamicMemoryEnabled $true -MinimumBytes 512MB -StartupBytes $MEMORY -MaximumBytes $MEMORY
	Add-VMDvdDrive -VMName $VMNAME -Path $InstallMediaPath
	$DVD = Get-VMDvdDrive -VMName $VMNAME
	Set-VMFirmware $VMNAME -FirstBootDevice $DVD
	# Set-VMKeyProtector -VMName $VMNAME -NewLocalKeyProtector
	# Enable-VMTPM -VMName $VMNAME
}

#
# Primary VM
#
$VMNAME = $VM1NAME
$VMDISK1 = $VM1DISK1
$VMDISK2 = $VM1DISK2
makeVM

## Print result
$VMInfo = Get-VM $VMName | Select-Object Name, State, @{Name="CPU"; Expression={$_.ProcessorCount}}, @{Name="MemoryGB"; Expression={[math]::Round($_.MemoryAssigned / 1GB, 2)}}, Version | Format-Table -AutoSize | Out-String
Write-Host "[I] VM Information:"
Write-Host $VMInfo

<#
$VHDXInfo = Get-VHD $VHDPath | Select-Object Path, @{Name="SizeGB"; Expression={[math]::Round(($_.Size / 1GB), 2)}}, VhdType | Format-Table -AutoSize | Out-String
Write-Host "[I] VHDX Information:"
Write-Host $VHDXInfo
#>

## Start VM and Connect to it
Start-VM -Name $VMNAME
vmconnect $env:COMPUTERNAME $VMNAME

#
# Secondary VM
#
$VMNAME = $VM2NAME
$VMDISK1 = $VM2DISK1
$VMDISK2 = $VM2DISK2
makeVM

$VMInfo = Get-VM $VMName | Select-Object Name, State, @{Name="CPU"; Expression={$_.ProcessorCount}}, @{Name="MemoryGB"; Expression={[math]::Round($_.MemoryAssigned / 1GB, 2)}}, Version | Format-Table -AutoSize | Out-String
Write-Host "[I] VM Information:"
Write-Host $VMInfo

Start-VM -Name $VMName
vmconnect $env:COMPUTERNAME $VMName
Pause
```

## Configuring NIC on VMs

```ps1
# Primary VM

## Disabling IPv6
Get-NetAdapter | Disable-NetAdapterBinding -ComponentID ms_tcpip6

## Configuring IP address & Gateway
netsh interface ip set address name="Ethernet"   static 192.168.137.11 255.255.255.0 192.168.137.1
netsh interface ip set address name="Ethernet 2" static      10.0.0.11 255.0.0.0

## Configuring DNS
netsh interface ipv4 set dns name="Ethernet" source=static addr=192.168.137.1 register=primary
```

```ps1
# Secondary VM

## Disabling IPv6
Get-NetAdapter | Disable-NetAdapterBinding -ComponentID ms_tcpip6

## Set IP address & Gateway
netsh interface ip set address name="Ethernet"   static 192.168.137.12 255.255.255.0 192.168.137.1
netsh interface ip set address name="Ethernet 2" static      10.0.0.12 255.0.0.0

## Configuring DNS
netsh interface ipv4 set dns name="Ethernet" source=static addr=192.168.137.1 register=primary
```
