---
layout:     post
title:      inputmethod switch
subtitle:   inputmethod
date:       2021-04-02
header-img: img/post-bg-desk.jpg
catalog:    true
tags:
    - Mac

---

# Mac 的自带输入法
中文很不友好，安装了Sougou 输入法之后，各种编辑器之间来回切换之后，状态很乱，当前明明切换成英文了
按键的时候却是中文，有时候明明是中文，输入又是英文，不断的切切切，几近崩溃...不断的切换，太烦人

# 自动切换输入法
找到一种办法，是保证在某些特定的几个应用里，始终默认是英文，其他的随便。这样每次回到那几个特定的编辑器时
最多切换一次，就保证满足需求

# 保险配置
最好再把下面这个选中
Mac - settings - keyboard - input source - Automatically switch to a document's input source


# 创建Python文件，内容如下

```
from AppKit import NSWorkspace, NSWorkspaceDidActivateApplicationNotification, NSWorkspaceApplicationKey
from Foundation import NSObject
from PyObjCTools import AppHelper
import ctypes
import ctypes.util
import objc
import CoreFoundation

# add your custom apps here, check the bundle id in /Application/xx.app/Contents/info.plist

ignore_list = [
    "com.googlecode.iterm2",
    "com.runningwithcrayons.Alfred-3",
    "com.jetbrains.intellij",
    "com.jetbrains.CLion",
    "com.jetbrains.WebStorm",
    "com.google.android.studio",
    "com.apple.Spotlight",
    "com.sublimetext.3"
]


carbon = ctypes.cdll.LoadLibrary(ctypes.util.find_library('Carbon'))

_objc = ctypes.PyDLL(objc._objc.__file__)

# PyObject *PyObjCObject_New(id objc_object, int flags, int retain)
_objc.PyObjCObject_New.restype = ctypes.py_object
_objc.PyObjCObject_New.argtypes = [ctypes.c_void_p, ctypes.c_int, ctypes.c_int]

def objc_object(id):
    return _objc.PyObjCObject_New(id, 0, 1)

# kTISPropertyLocalizedName
kTISPropertyUnicodeKeyLayoutData_p = ctypes.c_void_p.in_dll(carbon, 'kTISPropertyInputSourceIsEnabled')
kTISPropertyInputSourceLanguages_p = ctypes.c_void_p.in_dll(carbon, 'kTISPropertyInputSourceLanguages')
kTISPropertyInputSourceType_p = ctypes.c_void_p.in_dll(carbon, 'kTISPropertyInputSourceType')
kTISPropertyLocalizedName_p = ctypes.c_void_p.in_dll(carbon, 'kTISPropertyLocalizedName')
# kTISPropertyInputSourceLanguages_p = ctypes.c_void_p.in_dll(carbon, 'kTISPropertyInputSourceLanguages')

kTISPropertyInputSourceCategory = objc_object(ctypes.c_void_p.in_dll(carbon, 'kTISPropertyInputSourceCategory'))
kTISCategoryKeyboardInputSource = objc_object(ctypes.c_void_p.in_dll(carbon, 'kTISCategoryKeyboardInputSource'))


# TISCreateInputSourceList
carbon.TISCreateInputSourceList.restype = ctypes.c_void_p
carbon.TISCreateInputSourceList.argtypes = [ctypes.c_void_p, ctypes.c_bool]

carbon.TISSelectInputSource.restype = ctypes.c_void_p
carbon.TISSelectInputSource.argtypes = [ctypes.c_void_p]

carbon.TISGetInputSourceProperty.argtypes = [ctypes.c_void_p, ctypes.c_void_p]
carbon.TISGetInputSourceProperty.restype = ctypes.c_void_p

# carbon.TISCopyCurrentKeyboardLayoutInputSource.argtypes = []
# carbon.TISCopyCurrentKeyboardLayoutInputSource.restype = ctypes.c_void_p

carbon.TISCopyInputSourceForLanguage.argtypes = [ctypes.c_void_p]
carbon.TISCopyInputSourceForLanguage.restype = ctypes.c_void_p


def get_avaliable_languages():
    single_langs = filter(lambda x: x.count() == 1, \
        map(lambda x: objc_object(carbon.TISGetInputSourceProperty(CoreFoundation.CFArrayGetValueAtIndex(objc_object(s), x).__c_void_p__(), kTISPropertyInputSourceLanguages_p)), \
            range(CoreFoundation.CFArrayGetCount(objc_object(carbon.TISCreateInputSourceList(None, 0))))))
    res = set()
    map(lambda y: res.add(y[0]), single_langs)
    return res

def select_kb(lang):
    cur = carbon.TISCopyInputSourceForLanguage(CoreFoundation.CFSTR(lang).__c_void_p__())
    carbon.TISSelectInputSource(cur)

class Observer(NSObject):
    def handle_(self, noti):
        info = noti.userInfo().objectForKey_(NSWorkspaceApplicationKey)
        bundleIdentifier = info.bundleIdentifier()
        if bundleIdentifier in ignore_list:
            #print "found: %s active" % bundleIdentifier
            select_kb(u'en')


def main():
    nc = NSWorkspace.sharedWorkspace().notificationCenter()
    observer = Observer.new()
    nc.addObserver_selector_name_object_(
        observer,
        "handle:",
        NSWorkspaceDidActivateApplicationNotification,
        None
    )
    AppHelper.runConsoleEventLoop(installInterrupt=True)

main()
```

然后配置开机启动

我是放到 `~/startup/` 目录下

这个目录忘了当时是怎么配置的，后面更新出来

当然也可以按照 blog 上说的，用supervisor配置/编写plist加入开机启动/或者账户中心添加自动启动项目


参考文档

[github-gist-1](https://gist.github.com/tiann/f85e89bef4b6e9b83f2a)

[github-gist-2](https://gist.github.com/Khande/76f24ba90607fb5d54185bd8e4520de6)

[简书](https://www.jianshu.com/p/deab69015d2b)

[code leading](https://codeleading.com/article/8804569932/)
