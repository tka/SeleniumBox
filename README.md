# deuac.iso 
  
用來關掉 windows7 的 UAC, 從 ievms 的 Control ISO 改版出來, 只保留了移除 win7 UAC 的功能

# 設定 vmname

    vmname='IE8 - Win7'
# Import VM

   VBoxManage import $vmname.ova

# 建立乾淨的 Snapshot

   VBoxManage snapshot $vmname take init


# 關掉 UAC 並啟動 VM

deuac.iso 的來源是修改 ievms 的 Control ISO 建立方式做出來的, 只保留了移除 UAC 的功能

    VBoxManage storageattach $vmname --storagectl "IDE" --port 1 --device 0 --type dvddrive --medium deuac.iso 
    VBoxManage startvm $vmname
    while `VBoxManage showvminfo $vmname | grep -q 'running'` ; do; echo "waiting for disable UAC..."; sleep 1; done; echo "DONE\!"
    VBoxManage storageattach $vmname --storagectl "IDE" --port 1 --device 0 --type dvddrive --medium none 
  
# 更新 VBoxGuestAdditions

    VBoxManage storageattach $vmname --storagectl "IDE" --port 1 --device 0 --type dvddrive --medium VBoxGuestAdditions_4.3.10.iso
    VBoxManage startvm $vmname
    VBoxManage guestcontrol $vmname updateadditions --source VBoxGuestAdditions_4.3.10.iso
    VBoxManage guestcontrol $vmname exec --image c:/Windows/System32/shutdown.exe --username 'IEUser' --password 'Passw0rd!'  --verbose --wait-stderr --wait-stdout --wait-exit /r /t 0

# 複製程式到桌面

    VBoxManage guestcontrol $vmname mkdir c:/Users/IEUser/Desktop/selenium --username 'IEUser' --password 'Passw0rd!'  --verbose
    VBoxManage guestcontrol $vmname cp `pwd`/programs/ c:/Users/IEUser/Desktop/selenium/ --username 'IEUser' --password 'Passw0rd!' --recursive --verbose

# 安裝 java

    VBoxManage guestcontrol $vmname exec --image c:/Users/IEUser/Desktop/selenium/jre-7u51-windows-i586.exe --username 'IEUser' --password 'Passw0rd!'  --verbose --wait-stderr --wait-stdout --wait-exit /s

# 起動 Selenium

    VBoxManage guestcontrol $vmname exec --image c:/Users/IEUser/Desktop/selenium/01_start_hub.bat --username 'IEUser' --password 'Passw0rd!' --verbose
    VBoxManage guestcontrol $vmname exec --image c:/Users/IEUser/Desktop/selenium/02_start_node.bat --username 'IEUser' --password 'Passw0rd!' --verbose
