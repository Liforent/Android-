# 四大组件

# Debug技巧

- 如何定位到当前Activity是哪个？

  Logcat 清空所有过滤条件，输入 start u0。

  原理：system_server输出了ActivityRecord的相关日志。（所以不能是package: mine）