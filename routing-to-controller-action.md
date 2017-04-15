ASP.Net Core MVC 使用路由中间价处理请求的URL并映射到Action。路由定义在Startup.cs中或者特性[attribute]上。路由描述了哪些路径会匹配到Action，路由也用于生成URL并响应到客户端。

Action 是常规路由或特性路由。将路由放在Controller或Action中使其变为特性路由，更多信息请查看[特性路由](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#mixed-routing)

这篇文章会解释MVC和路由的相互作用，以及一个典型的MVC应用怎么使用路由特性。关于高级路由的信息请查看[路由](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing)

# 设置路由中间件
在你的`Configure`方法中你能看到类似的代码：
```csharp
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
```csharp
routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
```
### 路由模板
- `{controller=Home}`定义了`Controller`的默认值为`Home`
- `{action=Index}`定义了`Action`的默认值为`Index`
- `id?`定义了`id`为可选择的（可为空）

默认和可选路由参数不需要URL路径中匹配，有关路由模板的详细描述请参见[路由模板引用](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-reference)

`"{controller=Home}/{action=Index}/{id?}"`可以匹配URL路径`/`并且将产生路由值`{ controller = Home, action = Index }`。`controller`和`action`的值将使用默认值，`id`没有产生相应的值，因为路径中没有相应的段。MVC将使用这些路由值选择`HomeController`和`Index`Action.
```csharp
public class HomeController : Controller
{
  public IActionResult Index() { ... }
}
```
使用此控制器定义和路由模板，`HomeController.Index`将会执行下列任何URL路径。

- `Home/Index/17`
- `Home/Index`
- `Home`
- `/`

有一个方便的方法`UseMvcWithDefaultRoute`：
```csharp
app.UseMvcWithDefaultRoute();
```
可以用来替代
```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```
`UserMvc`和`UseMvcWithDefaultRoute`在中间件管道中添加了一个`RouterMiddleware`的实例。MVC不能与中间件直接交互，通过路由来处理请求。MVC通过`MvcRouteHandler`的实例连接到路由。`UseMvc`中的代码与下面的代码类似：
```csharp
var routes = new RouteBuilder(app);

// Add connection to MVC, will be hooked up by calls to MapRoute.
routes.DefaultHandler = new MvcRouteHandler(...);

// Execute callback to register routes.
// routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");

// Create route collection and add the middleware.
app.UseRouter(routes.Build());
```
`UseMvc`不能直接定义路由，它为路由集合中的属性路由添加点位符。重载`UseMvc(Action<IRouteBuilder>)`可以让以添加自己的路由并且支持属性路由。`UseMvc`和所有的变化都会为属性路由添加一个点位符。属性路由始终是可用的，不管如何配置`UseMvc`，`UseMvcWithDefaultRoute`定义默认路由并支持属性路由。属性路由的更多细节请查看[属性路由](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-ref-label)

## 传统的路由


