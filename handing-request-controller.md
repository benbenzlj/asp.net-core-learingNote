# 处理请求
## Controller辅助方法
### View
- 视图
    ```csharp
    return View(customer);
    ```
- Http状态码
    ```csharp
    return BadRequest();
    ```
- 格式化响应
    ```csharp
    return Json(customer);
    ```
- 内容协商响应
    action应该返回内容协商响应而不是一个对象，如（使用`OK`,`Created`,`CreateAndRoute`和`CreateAndAction`),例如：
    ```csharp
    return OK();
    return CreateAndRoute("routename",values,newobject);
    ```
- 跳转
    返回一个跳转到Action或者目的地的响应（使用`Redirect`,`LocalRedirect`,`RedirectToAction`和`RedirectToRoute`）。例如：
    ```csharp
    return RedirectToAction("Complete",new {id=123});
    ```
除以上方法外，action也可以简单的返回一个对象，在这种情况下，对象将基于客户端的请求被格式化。具体内容请参照[格式化响应](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/formatting)

