## Semi Automatic update

So far, we haven’t ended up fully automating the update. Full automation can also bring problems when the program is currently applying for changes if something has been sold locally or on the web. The problem is that if the quantity of a product is increased in either, then automation would see it as a sale in another.
## UpdateApp
```html
<h3>@Title</h3>
<table class="table table-striped table-dark rounded text-center">
    <thead>
        <tr>
            <th colspan="4">Update Application</th>
        </tr>
        <tr>
            <th colspan="2">Local</th>
            <th colspan="2">Web</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><LocalUpdate /></td>
            <td><LocalInfo /></td>
            <td><WebInfo/></td>
            <td><WebUpdate /></td>
        </tr>
    </tbody>
</table>
```
## Code section...
```csharp
    [Parameter]
    public string Title { get; set; }
```
## LocalUpdate Component
```html
@page "/localupdate"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config

<button class="btn-primary rounded" @onclick="GetDataLocal">
    Update
</button>
```
## Code section
```csharp
    DataClass dataclass = new DataClass();
    [Parameter]
    public string Title { get; set; }

    List<WebDbModel> webList;

    private async Task GetDataLocal()
    {
        if (webList != null)
        {
            webList.Clear();
        }
        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();
        string websql = $"CALL GetAllProducts; ";
        webList = await _dataweb.LoadWebData<WebDbModel,
            dynamic>(websql, new { },
            _config.GetConnectionString(dataclass.ConStringWeb));
        // Kutsutaan päivitys taski
        await UpdateData();
        stopwatch.Stop();
        Console.WriteLine("Time elapsed: {0:hh\\:mm\\:ss}", stopwatch.Elapsed);
    }
    // Päivitys taski
    private async Task UpdateData()
    {
        Console.WriteLine($"Processing local data...");
        foreach (var item in webList)
        {
            string sql = $"" +
                $"UPDATE Products " +
                $"SET Saldo = {Convert.ToString(item.Stock)} " +
                $"WHERE Webtuote = true " +
                $"AND Sku ='{Convert.ToString(item.SKU)}' " +
                $"AND Saldo > {Convert.ToString(item.Stock)};";

            await _datalocal.SaveData(sql, new { },
                _config.GetConnectionString(dataclass.ConStringLocal));
            Console.Write("R");
        }
        Console.WriteLine();
    }
```
