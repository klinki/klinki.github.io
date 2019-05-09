---
published: false
title: Hobby projekt v ASP.NET Core díl druhý - EF Core
---

Na samotném začátku projektu jsem přemýšlel, jaký zvolit ORM framework (a pokud vůbec nějaký používat). Mám rád [čistou architekturu](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) a chtěl jsem si vyzkoušet jak se dělá aplikace respektující tyto zásady v ASP.NETu. Jeden z mých hlavních požadavků na ORM knihovnu byl, aby se dala co nejsnáze odizolovat od doménových entit. Zjistil jsem, že tuto podmínku EF.Core celkem dobře splňuje (hlavně díky fluent konfiguraci) a tak jsem se rozhodl ho vyzkoušet.


## Instalace EF.Core as DB provideru
Prvním krokem byla instalace nuget balíčků `MySql.Data.EntityFrameworkCore` a příslušného databázového provideru, v mém případě `Pomelo.EntityFrameworkCore.MySql`.
Microsoft doporučuje nainstalovat také balíček `Microsoft.EntityFrameworkCore.Relational`. Ten by sice měl být nainstalovaný spolu s DB providerem, ale je lepší ho uvést jako přímou závslost, tak se k vám dostanou nejnovější verze nezávisle na poskytovateli DB provideru.

`Microsoft.EntityFrameworkCore.Tools.DotNet`

## Příkazová řádka
`dotnet ef`

## Reverse Engineering
Pokud máte již existující databázi, můžete si usnadnit práci použitím reverse engineeringu. EF Core tak vytvoří entity a mapování z existující databáze. Výsledkem bude složka obsahující soubor `*DbContext` a soubory entit.

`dontent ef dbcontext scaffold "Server=.\SQLEXPRESS;Database=SchoolDB;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models `

## Anotace vs. Fluent konfigurace
Mapování je možné konfigurovat pomocí anotací v třídách entit a nebo pomocí fluent konfigurace.

Fluent konfigurace je vhodnější protože je oddělená nezasahuje do doménových tříd. Pokud použijete rerverse engineering a vygenerujete entity z existující databáze, framework použije právě fluent konfiguraci. Ve výchozí konfiguraci vygeneruje jeden velký `DbContext` soubor, který bude obsahovat veškeré mapování pro všechny tabulky. Takto vygenerovaný soubor je velice velký a poměrně nepřehledný a tak jsem se rozhodl ho rozsekat na menší soubory s konfiguracemi pro jednotlivé entity.

Zde je ukázka jak může taková konfigurace vypadat:

```csharp
    public class UserConfiguration : IEntityTypeConfiguration<User>
    {
        public void Configure(EntityTypeBuilder<User> builder)
        {
            builder.ToTable("user");

            builder.Property(e => e.Id)
                .HasColumnName("id")
                .HasColumnType("int(11)")
                .ValueGeneratedOnAdd();

            builder.Property(e => e.Email)
                .IsRequired()
                .HasColumnName("email")
                .HasColumnType("varchar(255)");

            builder.Property(e => e.PasswordHash)
                .IsRequired()
                .HasColumnType("varchar(255)");

            builder.Property(e => e.RegisteredAt)
                .HasColumnName("registered_at")
                .HasColumnType("datetime");
        }
    }
```
```csharp
    public partial class GeneratedDbContext : IdentityDbContext<User, Role, int>
    {
        public IConfiguration Configuration { get; }

        public GeneratedDbContext()
        {
        }

        public GeneratedDbContext(DbContextOptions<GeneratedDbContext> options, IConfiguration configuration)
            : base(options)
        {
            Configuration = configuration;
        }

        public virtual DbSet<User> User { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            if (!optionsBuilder.IsConfigured)
            {
                optionsBuilder.UseMySql(Configuration.GetConnectionString("MySQLContext"), mysqlOptions => {

                });
            }
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
            // Načte všechny třídy implementující z assembly
            modelBuilder.ApplyConfigurationsFromAssembly(typeof(GeneratedDbContext).Assembly);
        }
    }
```

## Konfigurace
Do souboru `Startup.cs` metody `ConfigureServices(IServiceCollection services)` je potřeba přidat řádek `services.AddDbContext<GeneratedDbContext>;`.

Konfiguraci mám uloženou v souboru `appsettings.json` (a `appsettings.Development.json` pro vývojové prostředí). Pro produkční prostředí používám konfiguraci pomocí environment variables.

```json
{
  "ConnectionStrings": {
    "MySQLContext": "Server=localhost;port=3306;user=app;password=password;Database=app;GuidFormat=None"
  }
}
  ```

## Migrace dat
Entity framework poskytuje podporu pro migrace.
`dotnet ef migrations`
`dotnet ef database`

## Zdroje
[Entity Framework Tutorial](https://www.entityframeworktutorial.net/efcore/entity-framework-core.aspx)
