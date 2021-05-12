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
### Code section...
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
### Code section
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
## LocalInfo Component
```html
@page "/localinfo"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config
@if (webList == null)
{
    <button class="btn-primary rounded" @onclick="GetDataLocal">
        Get Info
    </button>
}
else
{

    if (localList.Count == 0)
    {
        <button class="btn-primary rounded" @onclick="GetDataLocal">
            Get Info
        </button>
        <p>
            Last check:
            <br />
            @DateTime.Now.ToString("dd.MM.yyyy HH:mm")
            <br />
            Nothing to update
            <hr />
            Click again to new info
        </p>

    }
    else
    {
        <button class="btn-warning rounded" @onclick="GetDataLocal">
            Get Info
        </button>
        <p>
            Last check:
            <br />
            @DateTime.Now.ToString("dd.MM.yyyy HH:mm")
            <br />
            Updatable: @localList.Count
            <hr />
            Click again to new info
        </p>
    }
}
```
### Code section
```csharp
    DataClass dataclass = new DataClass();
    [Parameter]
    public string Title { get; set; }
    List<WebDbModel> webList;
    List<LocalDbModel> localList = new List<LocalDbModel>();
    List<LocalDbModel> templocal;
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
        await GetData();
        stopwatch.Stop();


        Console.WriteLine("Time elapsed: {0:hh\\:mm\\:ss}", stopwatch.Elapsed);
    }

    private async Task GetData()
    {
        if (localList != null)
        {
            localList.Clear();
        }
        Console.WriteLine($"Loading...");
        foreach (var item in webList)
        {
            string sql = $"" +
                $"SELECT * FROM Products " +
                $"WHERE Webtuote = true AND Sku ='{Convert.ToString(item.SKU)}' " +
                $"AND Saldo > {Convert.ToString(item.Stock)};";
            templocal = await _datalocal.LoadData<LocalDbModel,
            dynamic>(sql, new { },
            _config.GetConnectionString(dataclass.ConStringLocal));

            if (templocal.Count != 0)
            {
                localList.AddRange(templocal);
                Console.Write($"I");
            }
            else
            {
                Console.Write($"O");
            }
        }
        Console.WriteLine();
        Console.WriteLine($"Add local updatable: {localList.Count}");
    }
```
