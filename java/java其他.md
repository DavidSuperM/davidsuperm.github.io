### java反编译工具
brew install --cask jd-gui
// 双击app，系统设置修改应用启动权限
open /Applications/JD-GUI.app


###  问题
为什么org.apache.commons.lang3.StringUtils源码中firstNonBlank方法中，子类对象赋值给父类对象，不会报错。如果赋值出来到自己代码是会正常报错的。
源码如下
```
@SafeVarargs
    public static <T extends CharSequence> T firstNonBlank(T... values) {
        if (values != null) {
            CharSequence[] var1 = values;
            int var2 = values.length;

            for(int var3 = 0; var3 < var2; ++var3) {
                T val = var1[var3];
                if (isNotBlank(val)) {
                    return val;
                }
            }
        }

        return null;
    }
```