# Entity Framework

## Code First

### DbContext

### DbSet

* Migrate Database

```PowerShell
# Package Manager Console 命令
# 注意如果DbContext所在Project & connection strings專案不同，需把Package Manager Console 右上角的Default Project改為DbContext所在Project，並且手動指定Connection Strings
Add-Migration Initial

# 小心會直接更新Database!
Update-Database -Verbose

# 日後更新可改用這方法產生Script，再到SSMS Run
Script-Migration 0 Initial
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

* Fluent API
* Data Annotations

* Value Conversions - to convert the enum to int when reading/writing to db
* [Data Seeding](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding) - to add the enum values in the db, in a migration

## Database First

### .edmx


## 參考資源
* [Enum to DB](https://stackoverflow.com/questions/50375357/how-to-create-a-table-corresponding-to-enum-in-ef-core-code-first)