---
title: BUG记录(持续更新)
date: 2020-05-29 14:50:26
tags:
---

### H5
- ios
Q: ios12+,当页面输入框聚焦时，切换页面，页面存在错位
<!-- more -->
A:
``` bash
el.addEventListener('blur', function () {
  const windowFocusHeight = window.innerHeight
  if (windowHeight === windowFocusHeight) return
  setTimeout(() => {
    const scrollHeight =
      document.documentElement.scrollTop||
      document.body.scrollTop||
      0
    window.scrollTo(0, Math.max(scrollHeight - 1, 0))
  }, 100)
})
```

Q：父元素设置overflow: hidden; 页面滚动会卡顿
A：设置 -webkit-overflow-scrolling: touch;

Q：IOS12及以下 页面高度为100% 设置固定定位 底部按钮滑动会消失
A：替换为绝对定位 滚动内容单独设置一个100%的盒子（或者使用flex布局，滚动元素设置flex: 1，固定元素设置高度，不使用定位）

Q：ios13 使用window.location.href 跳转到第三方页面 返回后页面不刷新
A：添加中间页面

- andriod
Q：输入框被激活时，键盘挡住输入框
A：
``` bash
if (/Android/gi.test(navigator.userAgent)) {
  window.addEventListener('resize', function () {
    window.setTimeout(function () {
      // document.activeElement.scrollIntoViewIfNeeded()
      document.activeElement.scrollIntoView(true)
    }, 0)
  })
}
```

Q: 安卓webView5.1.1系统白屏
A：第三方包es6语法未转换 使用transpileDependencies

Q: 弹窗蒙版穿透(Vue)
A：内容不可滑动 @touchmove.prevent 内容可滑动 @touchmove.self.prevent

Q: IOS13.4以下、部分安卓、chrome81以下版本拍照后图片会旋转
A：FIX：https://juejin.cn/post/6844904168075821069