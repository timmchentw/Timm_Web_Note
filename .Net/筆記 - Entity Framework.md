# Entity Framework

## Code First

### DbContext

### DbSet

* Migrate Database

```PowerShell
# Package Manager Console 命令
# 注意如果DbContext所在Project & connection strings專案不同，需把Package Manager Console 右上角的Default Project改為DbContext所在Project，並且手動指定Connection Strings
EntityFrameworkCore\Add-Migration Initial
# 如有多個DbContext檔案，指定其名稱
EntityFrameworkCore\Add-Migration Initial -context DataContextName

# 小心會直接更新Database!
EntityFrameworkCore\Update-Database -Verbose

# 日後更新可改用這方法產生Script，再到SSMS Run
EntityFrameworkCore\Script-Migration 0 Initial
EntityFrameworkCore\Script-Migration -From Initial -To XXX

# 這邊需要注意! Startup project會影響到EF Core取得Configuration的appsettings.josn路徑，如果Run語法失敗的話請更改Startup project再試看看
```

* Fluent API

```C#
//MyDbContext.cs

public partial class MyDbContext : DbContext
{
    public virtual DbSet<User> User { get; set; } = null!;

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            // 注意這邊的Connection string必須在Main project裡面DI註冊好 (services.AddDbContext)
            optionsBuilder.UseSqlServer("Name=ConnectionStrings:MyDb");
        }
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(entity =>
        {
            // 設定Table name
            entity.ToTable("user");
            
            // 型別為int、Not null且Name為"Id"的參數，EF將自動幫忙建立PK & Identity (流水號)
            entity.Property(e => e.Id).HasColumnName("id");
            
            // 設定Column型別
            entity.Property(e => e.Name)
                .HasMaxLength(10)
                .HasColumnName("name");
            
            entity.Property(e => e.RoleId)
               .HasColumnName("role_id");
            
            // 設定FoerignKey關聯其他表
            entity.HasOne(d => d.Role)
                .WithMany(p => p.Users)
                .HasForeignKey(d => d.RoleId)
                .OnDelete(DeleteBehavior.ClientSetNull)    // 注意Delete資料的行為限制
                .HasConstraintName("FK__user__role");
        });
    }
}
```

```C#
// User.cs
public partial class User
{
    public int Id { get; set; }
    public string? Name { get; set; }
    public int RoleId { get; set; }
    
    // Reference關聯物件
    public virtual Role Role { get; set; } = null!;
}

// Role.cs
public partial class Role
{
    public Role()
    {
        User = new HashSet<User>();
    }

    public int Id { get; set; }
    public string Name { get; set; } = null!;

    // Reference關聯物件
    public virtual ICollection<Users> Users { get; set; }
}
```

* Eager, Lazy, Implicit Load
  * Lazy (Deferred): 每次有需要再取得資料
  
    ```C#
    # Users參數必須是Virtual才能套用Lazy Loading
    var users = dbContext.Users;
    var targets = users.First().Role;
    ```

  * Eager: 一次取得所需關聯物件 (Include)
  
    ```C#
    var usersWithRole = dbContext.Users.Include(user => user.Role).First();
    ```

  * Implicit: 
  
    ```C#

    ```


* Fluent API
* Data Annotations

* Value Conversions - to convert the enum to int when reading/writing to db
* [Data Seeding](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding) - to add the enum values in the db, in a migration </br>
在DbContext檔案中定義預設Rows Data，適合如LOV、Mapping表等少改且需要作為Enum的Table

* 連動表 (Foreign Key Linked)

* AsQueryable </br>
  一般Select DB Repo返回的物件，屬於Lazy Loading，只有在ToList, First, Count...等執行指令時才會跑
  * 如果在Where條件中加入外部Function，會因為Linq to sql無法翻譯，而出現失效的狀況
* AsNoTracking </br>
  取得即時的資料

* Find </br>
  從Local DbContext取得資料，如果沒有才會去Query DB，類似Cache機制 (注意使用上可能會有Join Property為Null的情況)

   
* Id </br>
  設定Int Property為"Id"，EF Core將自動建立PK、Identity (流水號)

## Database First

### .edmx


## 參考資源
* [Enum to DB](https://stackoverflow.com/questions/50375357/how-to-create-a-table-corresponding-to-enum-in-ef-core-code-first)