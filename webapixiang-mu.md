# 1.新建webapi项目

在命令行中运行命令

```
dotnet new webapi --创建webapi项目
```

# 2.添加[Entity Framework Core InMemory](https://docs.microsoft.com/en-us/ef/core/providers/in-memory/)

在demo02.csproject 项目文件中添加

```csharp
<PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="1.1.1" />
```

添加保存后，visual studio code 会自动运行resotre命令，添加依赖

# 3.创建Models

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace demo02.Models
{
  public class  ToDoItem
  {
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public long  Key { get; set; }
    public string Name { get; set; }
    public bool IsComplete { get; set; }
  }
}
```

# 4.创建Context

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace demo02.Models {
    public class ToDoContext : DbContext {
        public ToDoContext (DbContextOptions options) : base (options) {

        }
        public DbSet<ToDoItem> ToDoItems { get; set; }
    }
}
```

# 5.创建IRepository

```
using System.Collections.Generic;

namespace demo02.Models
{
    public interface IToDoRepository
    {
        void Add(ToDoItem item);
        IEnumerable<ToDoItem> GetAll();
        ToDoItem Find(long key);
        void Remove(long key);
        void Update(ToDoItem item);
    }
}
```

# 6.创建Repository

```
using System;
using System.Collections.Generic;
using System.Linq;

namespace demo02.Models {
    public class ToDoRepository : IToDoRepository {
        private readonly ToDoContext _context;

        public ToDoRepository (ToDoContext context) {
            _context = context;
            Add (new ToDoItem () { Name = "Items1" });
        }

        public void Add (ToDoItem item) {
            _context.ToDoItems.Add(item);
            _context.SaveChanges();
        }

        public ToDoItem Find (long key) {
            return _context.ToDoItems.FirstOrDefault(d=>d.Key==key);
        }

        public IEnumerable<ToDoItem> GetAll () {
            return _context.ToDoItems.ToList();
        }

        public void Remove (long key) {
            var entity = _context.ToDoItems.First(d=>d.Key==key);
            _context.ToDoItems.Remove(entity);
            _context.SaveChanges();
        }

        public void Update (ToDoItem item) {
            _context.ToDoItems.Update(item);
            _context.SaveChanges();
        }
    }
}
```

# 7.在Startup中注入

```
public void ConfigureServices(IServiceCollection services)
        {
            //注入Context
            services.AddDbContext<ToDoContext>(opt=>opt.UseInMemoryDatabase());

            // Add framework services.
            services.AddMvc();
            //Controllers
            services.AddSingleton<IToDoRepository,ToDoRepository>();
        }
```

# 8.添加Controller

```
using System.Collections.Generic;
using Microsoft.AspNetCore.Mvc;
using demo02.Models;
using System.Linq;

namespace demo02.Controllers
{
    [Route("api/[controller]")]
    public class TodoController : Controller
    {
        private readonly IToDoRepository _todoRepository;

        public TodoController(IToDoRepository todoRepository)
        {
            _todoRepository = todoRepository;
        }
        [HttpGet]
        public IEnumerable<ToDoItem> GetAll()
        {
            return _todoRepository.GetAll();
        }
        [HttpGet("{id}",Name="GetTodo")]
        public IActionResult GetById(long id)
        {
            var item = _todoRepository.Find(id);
            if (item == null)
            {
                return NotFound();
            }
            return new ObjectResult(item);
    }
}
```

### 8.1 路由和URL路径

\[HttpGet\]特性标识指定响应Http Get 方法。每个方法的URL路径如下：

* 在控制的路由属性中获取模板字符串［Route\("api/\[controller\]"\)］
* 替换\[controller\]为路由名称，路由名称为路由类名去掉“Controller”后缀，在这个例子中，路由类名为“ToDoController"而路由的名称为"todo"。Asp.Net Core 中路由不区分大小写
* 如果［HttpGet］特性有一个模板字符串，添加到此路径。此示例不使用模板字符串。

在GetById方法中

```
[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(string id)
```

* {id}为变量点位符，当GetById被调用，url请求中的id参数为自动赋值到方法的id变量

* Name="GetTodo" 创建了一个命名路由，允许你在http 响应时链接到此路由。更多详情请参见[https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing)

### 8.2 返回值

GetAll 方法返回一个IEnumerable，MVC自动将对象序列化为JSON并输出到HTTP响应消息的Body中。返回码在这个方法中为200，假设这里有没有未处理的异常（未处理的异常会返回5xx的错误）

对比一下，GetById方法返回一般的**IActionResult**类型，它代表了大部分的返回类型，GetById方法有两个不同的返回结果。

* 如果找不到跟URL请求中ID匹配的项，返回404错误。通过返回**NotFound**来完成。
* 否则，方法返回200并且包含JSON响应。通过返回**ObjectResult**来完成。

# 9.启动APP

启动应用，visual studio环境按F5运行，Mac环境命令行中运行 **dotnet run.**启动后，访问[http://localhost:port/api/values，](http://localhost:port/api/values，)

# 10.实现其它的CRUD方法

## 10.1 Create

```
[HttpPost]
public IActionResult Create([FromBody] TodoItem item)
{
    if (item == null)
    {
        return BadRequest();
    }

    _todoRepository.Add(item);

    return CreatedAtRoute("GetTodo", new { id = item.Key }, item);
}
```

这是一个HTTP  POST 方法，通过\[HttpPost\]特性来标记，\[FromBody\]特性指明MVC通过获取http请求中的Body来获取to do Item的值。

CreateAtRout 返回一个201响应。这是在服务器上创建资源时返回的标准响应。CreateAtRoute也在HTTP响应头中添加了位置标签。位置标签指明了最近新创建资源的请求地址。

![](/assets/asp.net core/locationHeader.png)

你可以通过访问响应头的位置标签的内容来请求新创建的资源。回忆一下GetById创建了一个"GetToDo"的命名路由。

```
[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(string id)
```

## 10.2 Update

```
[HttpPut("{id}")]
public IActionResult Update(long id, [FromBody] TodoItem item)
{
    if (item == null || item.Key != id)
    {
        return BadRequest();
    }

    var todo = _todoRepository.Find(id);
    if (todo == null)
    {
        return NotFound();
    }

    todo.IsComplete = item.IsComplete;
    todo.Name = item.Name;

    _todoRepository.Update(todo);
    return new NoContentResult();
}
```

Update与Create类型，但是使用了HTTP PUT，响应为204（No Content）,根据HTTP规范，一个PUT请求要求客户端发送完整的更新实体，而不是部分的。如果要实现部分更新，请使用HTTP PATCH。

## 10.3 Delete

```
[HttpDelete("{id}")]
public IActionResult Delete(long id)
{
    var todo = _todoRepository.Find(id);
    if (todo == null)
    {
        return NotFound();
    }

    _todoRepository.Remove(id);
    return new NoContentResult();
}
```

响应为204（No Content）

`新加的`





