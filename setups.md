|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|

---

# Some Setups

Here are the main ...

### StarUp.cs services
```csharp
public void ConfigureServices(IServiceCollection services)
        {
            //// Razden
            //services.AddScoped<DialogService>();
            //services.AddScoped<NotificationService>();
            //services.AddScoped<TooltipService>();
            //services.AddScoped<ContextMenuService>();
            //// MatBlazor
            //services.AddMatBlazor();
            // Blazor
            services.AddRazorPages();
            services.AddServerSideBlazor();
            services.AddServerSideBlazor();
            // DataLibrary
            services.AddTransient<IDataAccess, DataAccess>();
            services.AddTransient<ILocalDataAccess, LocalDataAccess>();

        }
```
I’ve commented out a few of the services I tried but eventually found that I didn’t need them at least at this point. They are mostly free add-ons.
### Appsettins.json ConnectionStrings
```json
"ConnectionStrings": {

    "comment_1": "Connection strings here",
    "external_db": "Server-my-path-to-web-server;",
    "test_development": "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=C:\\Users\\Desctop\\test.mdb; User id=admin; Password=;",
    "test_server": "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=D:\\own\\folder\\path.mdb; User id=admin; Password=;",
    "real_server": "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=C:\\own\\folder\\path.mdb; User id=admin; Password=;"
  }
```
ConnectionStrings are added to the appsettings.json file

---
