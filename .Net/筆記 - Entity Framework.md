# Entity Framework

- [Entity Framework](#entity-framework)
  - [Code First](#code-first)
    - [DbContext](#dbcontext)
    - [DbSet](#dbset)
      - [Migrate Database](#migrate-database)
      - [Fluent API](#fluent-api)
      - [Eager, Lazy, Implicit Load](#eager-lazy-implicit-load)
    - [Data Setup](#data-setup)
    - [Commands (method-based query syntax)](#commands-method-based-query-syntax)
  - [Audit log](#audit-log)
  - [Database First](#database-first)
    - [.edmx](#edmx)
  - [參考資源](#參考資源)


## Code First

### DbContext

### DbSet

#### Migrate Database

- 務必先安裝`dotnet ef` & `Microsoft.EntityFrameworkCore.Design`，參考[MSDN](https://learn.microsoft.com/en-us/ef/core/cli/dotnet#installing-the-tools)

- CMD版本

```Bash
dotnet ef migrations add Migration_Name -c Target_DbContext --project Project_WithDbContext --startup-project Project_SetupTheConnectionString

dotnet ef migrations script FromMigrationName ToMigrationName
```


- Package manager console版本

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

#### Fluent API

- 口語化定義DB Table名稱、欄位、與Entity關係、Join關係、PK、FK、Constraint等等

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
            
            // 可以設定資料轉換
            entity.Property(e => e.RoleId)
               .HasColumnName("role_id")
               .HasConvertion(r => r.ToString());
            
            // 設定FoerignKey關聯其他表(1對多)
            entity.HasOne(d => d.Role)
                .WithMany(p => p.Users)
                .HasForeignKey(d => d.RoleId)
                .IsRequired(false)
                .OnDelete(DeleteBehavior.ClientSetNull)    // 注意Delete資料的行為限制
                .HasConstraintName("FK__user__role");

            entity.OwnsMany(d => d.Permissions)
                .
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
    public virtual ICollection<Permission> Permissions { get; set; } = new [];
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

- Owned
  - 用於定義某表附屬於特定表當中，不能單獨存在，一般適用於關聯表(如多對多)
    - 優點: 主表不用特別處理關連關係的IDs，而是可以直接關連整個Owned物件
    - 注意: 連query、update等等都不能單獨處理
    - 替換關聯關係，則需要用整個entity作替代 (可用Automapper由ID轉Entity)
  - 目標Entity加上 `[Owned]` attribute
  - 可定義一對一 `OwnsOne` 或一對多 `OwnsMany`

```C#
using Microsoft.EntityFrameworkCore;

public class MyDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Role> Roles { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // User為主表，關連多個UserRoles
        modelBuilder.Entity<User>()
            .OwnsMany(u => u.UserRoles, ur =>
            {
                // 附屬表提供關連主表Id
                ur.WithOwner().HasForeignKey(ur => ur.UserId);
                // 附屬表提供關連副表Id
                ur.HasOne(ur => ur.Role).WithMany().HasForeignKey(ur => ur.RoleId);
                ur.Property(ur => ur.Seq).ValueGeneratedOnAdd();
            });

                        entity.Navigation(x => )
    }
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }

    // Role的關連關係 (注意這邊沒有UserRoleId!)
    public ICollection<UserRole> UserRoles { get; set; }
}

public class Role
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public bool Active { get; set; }
}

[Owned]
public class UserRole
{
    public int Seq { get; set; }
    public int UserId { get; set; }
    public virtual Role Role { get; set; }
}

```

  - Automapper範例
    - 呈上的關連關係，可以透過AutoMapper，將關連IDs，轉換成關連Entities

    ```C#
    public class UserRoleUpdateDto
    {
        public int UserId { get; set; }
        public List<int> RoleIds { get; set; }  // 注意這邊只有ID
    }
    ```

    ```C#
    using AutoMapper;

    public class MappingProfile : Profile
    {
        public MappingProfile()
        {
            CreateMap<UserRoleUpdateDto, User>()
                .ForMember(
                  dest => dest.UserRoles, 
                  opt => opt.MapFrom(src => src.RoleIds.Select(roleId => new UserRole { Role = new Role { Id = roleId } })));
        }
    }
    ```

    ```C#
    public class UserService
    {
        private readonly MyDbContext _context;
        private readonly IMapper _mapper;

        public UserService(MyDbContext context, IMapper mapper)
        {
            _context = context;
            _mapper = mapper;
        }

        public async Task UpdateUserRoles(UserRoleUpdateDto dto)
        {
            var user = await _context.Users
                .Include(u => u.UserRoles)
                .ThenInclude(ur => ur.Role)
                .FirstOrDefaultAsync(u => u.Id == dto.UserId);

            if (user == null)
            {
                throw new Exception("User not found");
            }

            // 清除現有的 UserRoles
            user.UserRoles.Clear();

            // 使用 AutoMapper 更新 UserRoles
            _mapper.Map(dto, user);

            await _context.SaveChangesAsync();
        }
    }
    ```


#### Eager, Lazy, Implicit Load

  - Lazy (Deferred): 每次有需要再取得資料
  
    ```C#
    # Users參數必須是Virtual才能套用Lazy Loading
    var users = dbContext.Users;
    var targets = users.First().Role;
    ```

  - Eager: 一次取得所需關聯物件 (Include)
  
    ```C#
    var usersWithRole = dbContext.Users.Include(user => user.Role).First();
    ```

    - 或可於DbContext當中自動include

    ```C#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>()
            .Navigation(p => p.ShippingAddress)
            .AutoInclude();
    }
    ```

  - Implicit: 
  
    ```C#

    ```


### Data Setup

- Data Annotations
  - Value Conversions - to convert the enum to int when reading/writing to db
- [Data Seeding](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding) - to add the enum values in the db, in a migration </br>
在DbContext檔案中定義預設Rows Data，適合如LOV、Mapping表等少改且需要作為Enum的Table

- Id </br>
  設定Int Property為"Id"，EF Core將自動建立PK、Identity (流水號)

* AsQueryable </br>
  一般Select DB Repo返回的物件，屬於Lazy Loading，只有在ToList, First, Count...等執行指令時才會跑
  * 如果在Where條件中加入外部Function，會因為Linq to sql無法翻譯，而出現失效的狀況
* AsNoTracking </br>
  取得即時的資料
  
- 連動表 (Foreign Key Linked)
  - [Relationship](https://learn.microsoft.com/en-us/ef/core/modeling/relationships) (或稱Entity Reference)
    - 部分Join表可利用Relationship做綁定，配合Include指令做直接Join成單一entity的作法 (比如Entity A當中已有Property連動Entity B)

### Commands (method-based query syntax)

- AsQueryable </br>
  一般Select DB Repo返回的物件，屬於Lazy Loading，只要在ToList, First, Count...等執行指令時才會跑
  - 如果在Where條件中加入外部Function，會因為Linq to sql無法翻譯，而出現失效的狀況
  - 
- AsNoTracking </br>
  Tracking會暫存先前查詢的資料，以下情境可考慮:
    - Query:
      - Repo如果查詢相同Where條件，會返回之前查詢好的暫存內容，開啟後後可取得DB即時最新的資料
      - 唯讀且不須Save的資料，推薦開啟AsNoTracking，不進行追蹤而提升效能
    - Save: SaveChange時會比較Entity先前的改動進行追蹤，因此Query一次後、之後再Save就可以減少DB操作

    ```C#
    // Query暫存影響
    using (var context = new YourDbContext())
    {
        // 第一次查詢，實體會被追蹤
        var person1 = context.People.FirstOrDefault(p => p.Id == 1);
        Console.WriteLine(person1.Name); // 輸出：Original Name

        // 修改實體屬性
        person1.Name = "Updated Name";

        // 第二次查詢，相同的實體已經被追蹤，返回已追蹤的實體
        var person2 = context.People.FirstOrDefault(p => p.Id == 1);
        Console.WriteLine(person2.Name); // 輸出：Updated Name
    }

    ```

    ```C#
    // 開啟後比較
    using (var context = new YourDbContext())
    {
        // 查詢實體並禁用追蹤
        var person = context.People.AsNoTracking().FirstOrDefault(p => p.Id == 1);
        if (person != null)
        {
            // 修改實體屬性
            person.Name = "Updated Name";

            // 這些更改不會被保存，因為實體不被追蹤
            context.SaveChanges();
        }
    }
    ```

- Find </br>
  從Local DbContext取得資料，如果沒有才會去Query DB，類似Cache機制 (注意使用上可能會有Join Property為Null的情況)

- Aggregate
  - 優化效能使用的指令系列，只針對部分資料做計算
    - [MSDN](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ef/language-reference/method-based-query-syntax-examples-aggregate-operators)



## Audit log

- 為內建方法自動將Property改動存成Log到DB
- ABP Framework有整合此功能，請參考ABP note

## Database First

### .edmx


## 參考資源
- [Enum to DB](https://stackoverflow.com/questions/50375357/how-to-create-a-table-corresponding-to-enum-in-ef-core-code-first)
- [Owned Entity Types](https://learn.microsoft.com/en-us/ef/core/modeling/owned-entities)