# 在 mac 上用 vscode 開發 ASP.NET Core MVC

## 新增與開啟專案：

```shell
dotnet new mvc -o MvcFriends
code -r MvcFriends
```

## 初始化版本控制

```bash
git init
dotnet add gitignore
git add README.md
git add .
git commit -m "first commit"
```

若有遠端 repository，執行

```bash
git remote add origin https://github.com/.../xxx.git
git push -u origin master
```

## 執行

F5 執行，選擇 .NET Core

(信任 HTTPS 開發憑證：dotnet  dev-certs https - trust)

## 新增模型

Models/Friend.cs

```csharp
using System;
using System.ComponentModel.DataAnnotations;
namespace MvcFriends.Models
{
	public class Friend
	{
		public int Id { get; set; }
		public string Name { get; set; }
		public string Email { get; set; }
		public string Mobile { get; set; }
	}
}
```

## 新增NuGet Packages

安裝並更新 EF Core 的 CLI 命令工具 （**若已安裝過，應該可以不用再重複安裝**）

```bash
dotnet tool install --global dotnet-ef
dotnet tool update --global dotnet-ef
```

安裝 dotnet-aspnet-codegenerator 命令工具 （**若已安裝過，應該可以不用再重複安裝**）

```bash
dotnet tool install --global dotnet-aspnet-codegenerator
```

上面兩個工具安裝後可能需要將 .dotnet/tools 加到環境變數

執行新增工具及套件命令

```bash
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

## 新增 DatabaseContext.cs

在專案新增 Data 資料夾，建立 DatabaseContext.cs，它是 Entity Framework Core 負責對資料庫作業的物件

Data/DatabaseContext.cs

```csharp
using Microsoft.EntityFrameworkCore;
using MvcFriends.Models;
namespace MvcFriends.Data
{
  public class DatabaseContext : DbContext
  {
    public DatabaseContext(DbContextOptions<DatabaseContext> options) :
              base(options) {}
    public DbSet<Friend> Friends { get; set; }

  }
}
```

## 在 Startup.cs 註冊 DatabaseContext 服務

Startup.cs

```csharp
//... 略
using MvcFriends.Data;
using Microsoft.EntityFrameworkCore;

namespace MvcFriends
{
	public class Startup
	{
		public void ConfigureServices(IServiceCollection services)
		{
			services.AddControllersWithViews();
			services.AddDbContext<DatabaseContext>(options=>options.UseSqlite(
				Configuration.GetConnectionString("DatabaseContext")));
		}
	}
}
```

## 在 appsettings.json 新增 DatabaseContext 資料庫連線設定

appsettings.json

```csharp
{
  "Logging": {
    // ...
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DatabaseContext": "Data Source=MvcFriend.sqlite"
  }
}
```

## 建立Migration及產生資料庫

先停掉debugger

```bash
dotnet build
dotnet ef migrations add InitialDB
dotnet ef database update
```

p.s. 若在 ef migrations add 之後要刪除 migration，執行 ef migrations remove

## 新增 Controller, View

```bash
dotnet aspnet-codegenerator controller --controllerName FriendsController -outDir Controllers -async -namespace MvcFriends.Controllers -m Friend -dc DatabaseContext -udl
```

- —controllerName：指定控制器名稱
- -outDir：指定檔案輸出目錄
- -async：指產生非同步 Action
- -namespace：替控制器加上命名空間
- -m：使用的Model模型名稱
- -dc：要使用的DbContext名稱
- -udl：使用預設layout (use default layout)

查詢 dotnet aspnet-codegenerator 所有參數選項之作用

```bash
dotnet aspnet-codegenerator controller -h
```

---

## Seeding 建立種子資料

Data/DatabaseContext.cs

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
  modelBuilder.Entity<Friend>().HasData(
    new Friend { Id = 1, Name = "Mary", Email = "mary@gmail.com", Mobile = "0922-355822" },
    new Friend { Id = 2, Name = "David", Email = "david@gmail.com", Mobile = "0933-123456" },
    new Friend { Id = 3, Name = "Rose", Email = "rose@gmail.com", Mobile = "0955-888-163" }
  );
}
```

再次執行migration及資料庫更新

```bash
dotnet ef migrations add AddSeedData
dotnet ef database update
```

用資料庫管理工具查看 MvcFriend.sqlite 即可查看發現 Friends 資料表多了三筆資料

再按F5執行，瀏覽Friends/Index即可顯示資料