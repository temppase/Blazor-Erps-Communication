|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|

# Semi Automatic update

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
## WebUpdate Component
```html
@page "/webupdate"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config
@if (weblink == null || weblink.Count == 0)
{
    <button class="btn-primary rounded" @onclick="GetData">
        Update
    </button>
}
else
{
    <p>
        Last check: @DateTime.Now.ToString("dd.MM.yyyy HH:mm")
    </p>
}
```
### Code Section
```csharp
    DataClass dataclass = new DataClass();
    [Parameter]
    public string Title { get; set; }
    List<LocalDbModel> localList;
    List<WebDbModel> weblink = new List<WebDbModel>();
    List<WebDbModel> templist;
    protected override async Task OnInitializedAsync()
    {
        string sql = $"" +
                    $"SELECT * FROM Products " +
                    $"WHERE Webtuote = true " +
                    $"AND Saldo <> 100 " +
                    $"AND Saldo <> null;";
        localList = await _datalocal.LoadData<LocalDbModel,
        dynamic>(sql, new { },
        _config.GetConnectionString(dataclass.ConStringLocal));
    }
    private async Task GetData()
    {
        if (weblink.Count != 0)
        {
            weblink.Clear();
        }
        await OnInitializedAsync();
        Console.WriteLine();
        Console.WriteLine($"Loading web data...");
        foreach (var i in localList)
        {
            string websql = $"CALL GetHigherStock( @Stock, @SKU ); ";
            templist = await _dataweb.LoadWebData<WebDbModel,
                dynamic>(websql, new { @SKU = Convert.ToString(i.Sku), @Stock = Convert.ToString(i.Saldo) },
                _config.GetConnectionString(dataclass.ConStringWeb));
            if (templist.Count != 0)
            {
                weblink.AddRange(templist);
                Console.Write($"I");
            }
            else
            {
                Console.Write($"O");
            }
        }
        Console.WriteLine();
        Console.WriteLine($"The higher stock: {weblink.Count} ");
        if (weblink.Count != 0)
        {
            await UpdateData();
        }
        else
        {
            Console.WriteLine("Nothing to update");
        }
    }
    private async Task UpdateData()
    {
        //int n = 0;
        Console.WriteLine();
        Console.WriteLine($"Updating web data...");
        Console.ReadKey();
        foreach (var i in weblink)
        {
            //n++;
            //Console.WriteLine($"Päivitys nro: {n}");
            var s = int.Parse(i.Stock) - 1;
            string websql = $"CALL UpdateStock( @Stock, @ID ); ";
            await _dataweb.LoadWebData<WebDbModel,
                dynamic>(websql, new { @ID = Convert.ToString(i.ID), @Stock = Convert.ToString(s) },
                _config.GetConnectionString(dataclass.ConStringWeb));
        }
        await GetData();
    }
```
I had to implement a web update with ID number when I couldn't get a working MySQL procedure to be created that would combine the SKU and STOCK columns. In the WP database, they are in the same column.

## LocalInfo
```html
@page "/webinfo"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config
@if (localList == null)
{
    <button class="btn-primary rounded" @onclick="GetDataWeb">
        Get Info
    </button>

}
else
{
    if (weblink.Count == 0)
    {
        <button class="btn-primary rounded" @onclick="GetDataWeb">
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
        <button class="btn-warning rounded" @onclick="GetDataWeb">
            Get Info
        </button>
        <p>
            Last check:
            <br />
            @DateTime.Now.ToString("dd.MM.yyyy HH:mm")
            <br />
            Updatable: @weblink.Count
            <hr />Click again to new info
        </p>
    }
}
```
### Code section

```csharp
    public DataClass dataclass = new DataClass();
    [Parameter]
    public string Title { get; set; }
    List<LocalDbModel> localList;
    // Jos taski käyttää listaa loopissa se pitää initalisoida näin
    List<WebDbModel> weblink = new List<WebDbModel>();
    List<WebDbModel> templist;
    // First feching the data from the local db
    private async Task GetDataWeb()
    {
        if (localList != null)
        {
            localList.Clear(); // Vanhan datan tyhjennys
        }
        var stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();
        string sql = $"" +
                    $"SELECT * FROM Products " +
                    $"WHERE Webtuote = true " +
                    $"AND Saldo <> 100 " +
                    $"AND Saldo <> null;";
        localList = await _datalocal.LoadData<LocalDbModel,
        dynamic>(sql, new { },
        _config.GetConnectionString(dataclass.ConStringLocal));
        // The metod call that fech the related gata from the web
        await GetData();
        stopwatch.Stop();
        Console.WriteLine($"Time elapsed: {stopwatch.Elapsed:hh\\:mm\\:ss}");
    }


    private async Task GetData()
    {
        if (weblink != null)
        {
            weblink.Clear(); // Vanhan datan tyhjennys
        }
        Console.WriteLine();
        Console.WriteLine($"Loading web data...");
        foreach (var i in localList)
        {
            string websql = $"CALL GetHigherStock( @Stock, @SKU ); ";
            templist = await _dataweb.LoadWebData<WebDbModel,
                dynamic>(websql, new { @SKU = Convert.ToString(i.Sku), @Stock = Convert.ToString(i.Saldo) },
                _config.GetConnectionString(dataclass.ConStringWeb));
            if (templist.Count != 0)
            {
                // Tässä käytetään listaa joka initalisoitiin erilailla.
                weblink.AddRange(templist);
                Console.Write($"I");

            }
            else
            {
                Console.Write($"O");

            }

        }
        Console.WriteLine();
        Console.WriteLine($"The higher stock: {weblink.Count} ");

    }
```
