Mac上做Android开发，使用Android真机无法访问到，解决方案如下

$  system_profiler SPUSBDataType 
找到  Vendor ID: 0x22da 
vi ~/.android/adb_usb.ini  
输入 0x22da 前面不能有空行以及空格 
$  cd /Users/wangjiguang/Library/Android/sdk/platform-tools/ 
$  ./adb kill-server 
$  ./adb start-server 
$  ./adb devices 
List of devices attached  
0123456789ABCDEF    device

再运行后，一切正常

