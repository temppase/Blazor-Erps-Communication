# Manual Update

|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|

First the index page...
```html
@page "/man/update"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config

<h3>Manual update</h3>
<OrderDate />
<hr />

<ManualUpdateTable />

<Footer Title=@Title />
```
### Code section
```csharp
    [Parameter]
    public string Title { get; set; } = "| Manual update ";
```
Nothing much...

## ManualUpdateTable Component
```html
@page "/manualupdate"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config
<h3 class="p-3
    mb-2
    bg-dark
    text-white
    text-center
    font-weight-bold">
    Manual update version 4.5
</h3>
    <OrderDate />
<br />
    @if (WebDataFiltered.Count != 0)
    {


        <CustomTable Items="WebDataFiltered">
            <TableHeader>
                <GridColum ColumTitle="SKU"
                            OnSearchTextChange="@(e =>SearchTermChanged(e, "SKU"))" />


                <GridColum ColumTitle="NAME"
                            OnSearchTextChange="@(e =>SearchTermChanged(e, "NAME"))" />
                <th>Stock</th>
            </TableHeader>
            <RowTemplate Context="p">
                <a href="@($"manualupdate/{BtnValueSku = p.SKU.Replace("/","*").Replace(".","$")}")">
                    @p.SKU
                </a>
                <td>
                    @p.Name
                </td>
                <td>@p.Stock</td>
            </RowTemplate>
        </CustomTable>
        <p style="padding:2%">Count: @WebDataFiltered.Count</p>

    }
    else
    {


        <h4>No Results</h4>
        <button class="btn btn-primary" 
                onClick="window.location.reload(true)">
            Refresh
        </button>
    }



<Footer Title="Manual update" />
```
### Code section
```csharp
    public DataClass dataclass = new DataClass();
    public List<LocalDbModel> LocalData;
    public WebDbModel Products { get; set; }
    public List<WebDbModel> WebData { get; set; }
    public List<WebDbModel> WebDataFiltered { get; set; }
    [Parameter]
    public string BtnValueSku { get; set; }
    [Parameter]
    public string BtnValueName { get; set; }


    protected override async Task OnInitializedAsync()
    {
        Console.WriteLine($"Loading web data...");
        // Change this procedure to update
        string websql = $"CALL GetAllProducts(); ";
        WebData = await _dataweb.LoadWebData<WebDbModel,
            dynamic>(websql, new { },
            _config.GetConnectionString(dataclass.ConStringWeb));
        Console.WriteLine(WebData.Count);
        if (WebDataFiltered == null)
        {
            WebDataFiltered = WebData;
        }
    }
    private void SearchTermChanged(ChangeEventArgs changeEventArgs, string columTitle)
    {

        string searchText = changeEventArgs.Value.ToString();
        Console.WriteLine(searchText);
        if (columTitle == "NAME")
        {
            WebDataFiltered = WebData.Where(p => p.Name.Contains
        (searchText, StringComparison.OrdinalIgnoreCase)).ToList();
        }
        if (columTitle == "SKU")
        {
            WebDataFiltered = WebData.Where(p => p.SKU.Contains
        (searchText)).ToList();
        }
    }

```
This component contains quite a lot of stuff. I’m not sure what would be the best way to unpack it but I try to open it as much as possible. 
I give some components their own page...


SIteJoin component you won't see in code... but it's in the hyperlink.
## SiteJoin Component
```html

@page "/manualupdate/{BtnValueSku}"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config

<br />

<div class="row">
    <div class="col-6">
        <h5>
            Local:
        </h5>
        
            @if (LocalDataFiltered != null)
            {
                @foreach (var l in LocalDataFiltered)
                {
                    <p style="font-weight:900">
                        @l.Valmistaja @l.Tuote
                    </p>
                }
            }


        @if (LocalDataFiltered is null)
        {
            <p style="text-indent:10%; font-style:italic">Loading...</p>
        }
        else if (LocalDataFiltered.Count == 0)
        {
            <p style="text-indent:10%; font-style:italic">No results...</p>
        }
        else
        {
            @foreach (var l in LocalDataFiltered)
            {
                <p style="text-indent:10%">In stock: @l.Saldo</p>
            }
        }
        <h5>
            Web:        
        </h5> 
        <p style="font-weight:900">
            @if (WebDataFiltered != null)
            {
                @foreach (var w in WebDataFiltered)
                {
                    @w.Name
                }
            }
        </p>
        @if (WebDataFiltered is null)
        {
            <p style="text-indent:10%; font-style:italic">Loading...</p>
        }
        else if (WebDataFiltered.Count == 0)
        {
            <p style="text-indent:10%; font-style:italic">No results...</p>
        }
        else
        {
            @foreach (var w in WebDataFiltered)
            {
                <p style="text-indent:10%">In stock: @w.Stock</p>
            }
        }
    </div>
    <div class="col-4">
            <SkuUpdate Sku=@Sku/>

    </div>
</div>
<Footer Title="Manual update SKU" />
```
### Code section
```csharp
    public DataClass dataclass = new DataClass();
    public List<LocalDbModel> LocalData { get; set; }
    public List<WebDbModel> WebData { get; set; }
    public List<WebDbModel> WebDataFiltered { get; set; }
    public List<LocalDbModel> LocalDataFiltered { get; set; }
    [Parameter]
    public string BtnValueSku { get; set; }
    [Parameter]
    public string Sku { get; set; }
    protected override async Task OnInitializedAsync()
    {
        Sku = BtnValueSku.Replace("*", "/").Replace("$",".");
        Console.WriteLine($"Loading web data...");
        // Change this procedure to update
        string websql = $"CALL GetAllProducts(); ";
        WebData = await _dataweb.LoadWebData<WebDbModel,
            dynamic>(websql, new { },
            _config.GetConnectionString(dataclass.ConStringWeb));
        Console.WriteLine(WebData.Count);
        await GetLocalData();
        WebSku();
        LocalSku();

    }
    private async Task GetLocalData()
    {
        Console.WriteLine(Sku);
        Console.WriteLine($"Loading local data...");
        string localsql = $"" +
                $"SELECT * FROM Products " +
                $"WHERE Webtuote = true AND Sku ='{Sku}';";
        LocalData = await _datalocal.LoadData<LocalDbModel,
        dynamic>(localsql, new { },
        _config.GetConnectionString(dataclass.ConStringLocal));
        Console.WriteLine(LocalData.Count);

    }
    private void WebSku()
    {
        Console.WriteLine(Sku);
        WebDataFiltered = WebData.Where(p => p.SKU.Equals
        (Sku)).ToList();
    }
    private void LocalSku()
    {
        Console.WriteLine(Sku);
        LocalDataFiltered = LocalData.Where(p => p.Sku.Equals
        (Sku)).ToList();
    }
```
## SkuUpdate Component
```html

@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config

<form>
    <input type="button" 
           class="form-group" 
           aria-label="Small"
           aria-describedby="inputGroup-sizing-sm"
           value="Sku: @Sku"
           @oninput="@OnInputChange">
    <br />
    <label> New amount:</label>
    <input type="text" 
           class="form-group" 
           aria-label="Small"
           aria-describedby="inputGroup-sizing-sm"
           @bind="@Stock" 
           @bind:event="oninput">
    <div class="form-group">
        <button class="btn btn-primary" 
                id="inputGroup-sizing-sm"
                @onclick="ManualUpdate" 
                type="submit">
            Update
        </button>
    </div>

</form>
@if (ErrMesg != null)
{
    <p>@ErrMesg</p>
}

```
### Code section
```csharp
    DataClass dataclass = new DataClass();
    [Parameter]
    public string Sku { get; set; }
    [Parameter]
    public string Stock { get; set; }
    [Parameter]
    public int Id { get; set; }
    [Parameter]
    public string ErrMesg { get; set; }
    public void Errormesage()
    {
        ErrMesg = "Not valid data";
    }

    [Parameter]
    public EventCallback<string> ValueChanged { get; set; }
    public async Task OnInputChange(ChangeEventArgs args)
    {
        Sku = (string)args.Value;
        await ValueChanged.InvokeAsync(Sku);
    }
    List<WebDbModel> WebSearch;
    private async Task ManualUpdate()
    {
        if (Stock == null || Stock.Length > 6)
        {
            Errormesage();
        }
        else
        {
            Console.WriteLine();
            Console.WriteLine($"Loading web data by sku {Sku}...");

            string websql = @"CALL GetBySKU( @SKU);";
            WebSearch = await _dataweb.LoadWebData<WebDbModel,
                dynamic>(websql, new { @SKU = $"{Sku}" },
                _config.GetConnectionString(dataclass.ConStringWeb));

            Console.WriteLine(websql);
            Console.WriteLine($"Manual update...");

            await ManualUpdateWeb();
            await ManualUpdateLocal();
            StateHasChanged();
        }
    }


    private async Task ManualUpdateWeb()
    {
        Console.WriteLine();
        Console.WriteLine($"Web...");
        if (WebSearch != null)
        {
            foreach (var i in WebSearch)
            {
                Console.WriteLine($"Web start at value: {Convert.ToString(i.ID)}...");
                string websql = $"CALL UpdateStock( @Id, @Stock ); ";
                Console.WriteLine(websql);
                await _dataweb.LoadWebData<WebDbModel,
                    dynamic>(websql, new { @Stock = Stock, @Id = Convert.ToString(i.ID) },
                    _config.GetConnectionString("avecomfi_db"));
            }


        }
        else
        {
            Console.WriteLine();
            Console.WriteLine($"Web have no value...");
        }

    }
    private async Task ManualUpdateLocal()
    {
        Console.WriteLine();
        Console.WriteLine($"Local...");
        if (Sku != null && Sku != "" && Stock != "")
        {
            Console.WriteLine($"Local start at value: {Sku}...");
            string sql = $"" +
            $"UPDATE Products " +
            $"SET Saldo = {Stock} " +
            $"WHERE Webtuote = true " +
            $"AND Sku ='{Sku}';";
            Console.WriteLine(sql);
            await _datalocal.SaveData(sql, new { },
                _config.GetConnectionString(dataclass.ConStringLocal));

        }
        else
        {
            Console.WriteLine($"Local no value...");
        }
    }
```

I put the controllers I have used on their own page so it doesn’t get too confusing.
