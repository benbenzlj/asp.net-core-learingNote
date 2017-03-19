# 1.新建webapi项目

在命令行中运行命令

```
dotnet new webapi --创建webapi项目
```

# 2.添加[Entity Framework Core InMemory](https://docs.microsoft.com/en-us/ef/core/providers/in-memory/)

在demo02.csproject 项目文件中添加

```
<PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="1.1.1" />
```

添加保存后，visual studio code 会自动运行resotre命令，添加依赖

# 3.创建Models

```
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







