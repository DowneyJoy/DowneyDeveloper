### 若windows电脑忘记密码

- 长按Shift重启电脑

- 选择高级选项---命令行选项
- 输入 copy cmd.exe C:\windows\system32\osk.exe【若复制失败，选择其他盘符继续测试例如：copy cmd.exe D:\windows\sytem32\osk.exe，复制成功后点击y】关闭命令行
- 右下角打开键盘映射：输入netplwiz
- 选择用户修改密码

跳过windows密码直接进入

### 使用 netplwiz 命令（最推荐，最安全）
这是微软官方内置的方式，它并非删除密码，而是让系统自动用你保存的凭据登录，账户密码本身并未被修改或擦除-
￼
。
操作步骤：

1. 关闭 Windows Hello 强制要求（仅限微软账户）：按 Win + I 打开设置，进入 “账户” > “登录选项”，关闭 “为了提高安全性，仅允许对此设备上的 Microsoft 账户使用 Windows Hello 登录” 的开关。
2. 调出运行窗口：按 Win + R，输入 netplwiz，回车
3. 取消密码验证：在“用户账户”窗口中，选中你的账户，取消勾选 “要使用本计算机，用户必须输入用户名和密码”，然后点击“应用”
4. 确认自动登录密码：在弹出的对话框中，输入两次你当前的账户密码（微软账户需输入完整邮箱地址作为用户名），点击“确定”
如果找不到勾选框，说明系统隐藏了该选项，需先修改注册表：打开 regedit，定位到 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\PasswordLess\Device，将 DevicePasswordLessBuildVersion 的值改为 0，重启电脑后再从第1步重新操作即可

```C#
using System.Management;
using UnityEngine;

public class WmiRemoteShutdown : MonoBehaviour
{
    public string targetIP = "192.168.1.100";
    public string username = "Administrator";
    public string password = "your_password";

    public void ShutdownViaWmi()
    {
        try
        {
            ConnectionOptions options = new ConnectionOptions
            {
                Username = username,
                Password = password,
                Authentication = AuthenticationLevel.PacketPrivacy,
                Impersonation = ImpersonationLevel.Impersonate
            };

            ManagementScope scope = new ManagementScope($"\\\\{targetIP}\\root\\cimv2", options);
            scope.Connect();

            ObjectQuery query = new ObjectQuery("SELECT * FROM Win32_OperatingSystem");
            ManagementObjectSearcher searcher = new ManagementObjectSearcher(scope, query);

            foreach (ManagementObject mo in searcher.Get())
            {
                // 6 = 强制关机, 2 = 重启
                ManagementBaseObject inParams = mo.GetMethodParameters("Win32Shutdown");
                inParams["Flags"] = 6; 
                mo.InvokeMethod("Win32Shutdown", inParams, null);
            }
            Debug.Log("WMI关机指令已发送");
        }
        catch (System.Exception e)
        {
            Debug.LogError($"WMI关机失败: {e.Message}");
        }
    }
}
```