## Manual Update
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
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config

@if (productListFiltered.Count != 0)
{


    <CustomTable Items="productListFiltered">
        <TableHeader>
            <CascadingValue Name="ManualUpdateTable" Value="BtnValueSku">
                <GridColum ColumTitle="SKU"
                           OnSearchTextChange=
                           "@(e =>SearchTermChanged(e, "SKU"))" 
                           />
            </CascadingValue>
            <CascadingValue Name="ManualUpdateTable" Value="BtnValueName">
                <GridColum ColumTitle="NAME"
                           OnSearchTextChange=
                           "@(e =>SearchTermChanged(e, "NAME"))" 
                           />
                <th>Stock</th>
            </CascadingValue>
        </TableHeader>
        <RowTemplate Context="p">
            <td>
                <button @onclick="(e => BtnValueSku = p.SKU)">
                    @p.SKU
                </button>
            </td>
            <td>
                <button @onclick="(e => BtnValueName = p.Name)">
                    @p.Name
                </button>
            </td>
            <td>@p.Stock</td>
        </RowTemplate>
    </CustomTable>
    <p>Count: @productListFiltered.Count Sku: @BtnValueSku</p>
    

}
else
{


    <h4>No Results</h4>
    <button class="btn btn-primary" onClick="window.location.reload(true)">
        Refresh
    </button>
}
<CascadingValue Name="Sku" Value="BtnValueSku">
    <SkuUpdate/>
    <LocalSku/>
</CascadingValue>
```
### Code section
```csharp
    public WebDbModel products { get; set; }
    public List<WebDbModel> productList { get; set; }
    public List<WebDbModel> productListFiltered { get; set; }
    [Parameter]
    public string BtnValueSku { get; set; }
    [Parameter]
    public string BtnValueName { get; set; }


    protected override async Task OnInitializedAsync()
    {
        Console.WriteLine($"Loading web data...");
        // Change this procedure to update
        string websql = $"CALL GetAllProducts(); ";
        productList = await _dataweb.LoadWebData<WebDbModel,
            dynamic>(websql, new { },
            _config.GetConnectionString("avecomfi_db"));
        Console.WriteLine(productList.Count);
        if (productListFiltered == null)
        {
            productListFiltered = productList;
        }
    }
    private void SearchTermChanged(ChangeEventArgs changeEventArgs, string columTitle)
    {

        string searchText = changeEventArgs.Value.ToString();
        Console.WriteLine(searchText);
        if (columTitle == "NAME")
        {
            productListFiltered = productList.Where(p => p.Name.Contains
            (searchText, StringComparison.OrdinalIgnoreCase)).ToList();
        }
        if (columTitle == "SKU")
        {
            productListFiltered = productList.Where(p => p.SKU.Contains
            (searchText)).ToList();
        }
    }

```
This component contains quite a lot of stuff. I’m not sure what would be the best way to unpack it but I try to open it as much as possible. 
I'll start with the easiest way...but perhaps most importantly...

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
