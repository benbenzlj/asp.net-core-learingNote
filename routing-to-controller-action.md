ASP.Net Core MVC 使用路由中间价处理请求的URL并映射到Action。路由定义在Startup.cs中或者特性[attribute]上。路由描述了哪些路径会匹配到Action，路由也用于生成URL并响应到客户端。

Action 是常规路由或特性路由。将路由放在Controller或Action中使其变为特性路由，更多信息请查看[特性路由](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#mixed-routing)

这篇文章会解释MVC和路由的相互作用，以及一个典型的MVC应用怎么使用路由特性。关于高级路由的信息请查看[路由](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing)

# 设置路由中间件
在你的`Configure`方法中你能看到类似的代码：
```c#
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```
里面的`UseMvc`,`MapRoute`用来创建一个单一的路由，我们将之称为`默认`路由。更多的MVC应用使用与默认路由类似的路由模板。

路由模板`"{controller=Home}/{action=Index}/{id?}"`将匹配类似`/Products/Details/5`的URL路径，并且根据标志提取路由中的值`{ controller = Products, action = Details, id = 5 }`。MVC将尝试定位一个名为`ProductsController`的Controller，并且运行Action`Details`：
```c#
public class ProductsController : Controller
{
   public IActionResult Details(int id) { ... }
}
```
注意这个例子，当调用Action时，模型绑定将使用`id=5`的值设置`id`参数的值为`5`。更多详情请查看[模型绑定](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding)

## 使用`默认`路由：
```c#
routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
```
### 路由模板
- `{controller=Home}`定义了`Controller`的默认值为`Home`
- `{action=Index}`定义了`Action`的默认值为`Index`
- `id?`定义了`id`为可选择的（可为空）

默认和可选路由参数不需要URL路径中匹配，有关路由模板的详细描述请参见[路由模板引用](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-reference)

