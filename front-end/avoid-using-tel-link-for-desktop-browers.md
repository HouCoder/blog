# 避免在桌面端浏览器中使用 tel 链接

对于移动端的 `a` 标签来说， `href` 中使用 `tel` 标记可以方便的让用户拨打电话，最近在开发桌面端时也使用了相同的手段展示电话号码，在 Mac 下测试点击链接 macOS 弹出对话框询问是否拨打电话，自以为这对于桌面端的用户体验也是一个提升，开发完成后测试时发现 Windows 系统中浏览器对于这个属性的支持并不好，用户点击链接后因为很多浏览器不能识别 `tel` 协议跳转到浏览器的出错页面：

![](./avoid-using-tel-link-for-desktop-browers/bs_win7_Firefox_52.0.jpeg)

在主流版本的 Windows 上针对主流浏览器测试后发现对于 `tel` 链接只有 Windows 8 以上的 IE 与 Edge 支持，点击后会弹窗询问用哪个应用打开该协议，其它的浏览器要么点击无反应要么跳转到出错页面。下面是测试完后整理的测试结果：

| Browser    | OS          | 是否生成链接 | 点击后                         |
|------------|-------------|--------------|--------------------------------|
| IE 8       | Windows 7   | 是           | 跳转至错误页面                 |
| IE 11      | Windows 7   | 是           | 跳转至错误页面                 |
| Chrome 55  | Windows 7   | 是           | 无响应                         |
| Firefox 50 | Windows 7   | 是           | 跳转至错误页面                 |
| IE10       | Windows 8   | 是           | 弹窗询问使用什么应用打开该链接 |
| Chrome 55  | Windows 8   | 是           | 无响应                         |
| Firefox 50 | Windows 8   | 是           | 跳转至错误页面                 |
| IE 11      | Windows 8.1 | 是           | 弹窗询问使用什么应用打开该链接 |
| Chrome 55  | Windows 8.1 | 是           | 无响应                         |
| Firefox 50 | Windows 8.1 | 是           | 跳转至错误页面                 |
| IE 11      | Windows 10  | 是           | 弹窗询问使用什么应用打开该链接 |
| EDGE       | Windows 10  | 是           | 弹窗询问使用什么应用打开该链接 |
| Chrome 55  | Windows 10  | 是           | 无响应                         |
| Firefox 50 | Windows 10  | 是           | 跳转至错误页面                 |
