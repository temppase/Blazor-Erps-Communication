|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|

---

# Orders Info

## OrderData Component
```html
@page "/orderinfo"
@inject ILocalDataAccess _datalocal
@inject IDataAccess _dataweb
@inject IConfiguration _config

@if (OrderListFiltered.Count != 0)
{
    <button class="btn btn-outline-info"
            type="button"
            data-toggle="collapse"
            data-target="#OrderDate"
            aria-expanded="false"
            aria-controls="OrderDate">
        Order info
    </button>

    <div class="collapse"
         id="OrderDate">

        <br />
        <div>
            <CustomTable Items="OrderListFiltered">
                <TableHeader>
                    <GridColumCalendar ColumTitle="Date"
                                       OnSearchDateChange="@(e =>SearchDateChanged(e, "Date"))" />
                    <GridColum ColumTitle="Ordered Items"
                               OnSearchTextChange="@(e =>SearchTermChanged(e, "Ordered Items"))" />
                    <GridColum ColumTitle="Status" 
                               OnSearchTextChange="@(e =>SearchTermChanged(e, "Status"))" />
                </TableHeader>
                <RowTemplate Context="p">
                    <td>
                        @p.PurchaseDate.ToString("dd.MM.yyyy")
                    </td>
                    <td>
                        @p.ItemsOrdered
                    </td>
                    @if (p.PurchaseStatus != "Completed")
                        {
                        <td style="background-color:red;">@p.PurchaseStatus</td>
                        }
                        else
                        {
                        <td>@p.PurchaseStatus</td>
                        }

                </RowTemplate>
            </CustomTable>
            <p>
                Count: @OrderListFiltered.Count | Check the percentages:
                <button class="btn btn-outline-info" @onclick="@ProCount">@Procentage %</button>
            </p>
        </div>
    </div>

}
else
{
    <h4>No Results</h4>
    <button class="btn btn-primary"
            onClick="window.location.reload(true)">
        Refresh
    </button>
}
```
### Code section
```csharp
    DataClass dataclass = new DataClass();
    public OrderInfoModel Orders { get; set; }
    [Parameter]
    public double Procentage { get; set; }

    private void ProCount()
    {
        Procentage = ((double)OrderListFiltered.Count / OrderList.Count) * 100;
        Procentage = Math.Round(Procentage, 2);
        Console.WriteLine(Procentage);
    }

    public List<OrderInfoModel> OrderList { get; set; }
    public List<OrderInfoModel> OrderListFiltered { get; set; }
    public List<OrderInfoModel> OrderListFilteredStatus { get; set; }
    protected override async Task OnInitializedAsync()
    {
        Console.WriteLine();
        Console.WriteLine($"Loading GetOrders...");
        string websql = $"CALL GetOrders( ); ";
        OrderList = await _dataweb.LoadWebData<OrderInfoModel,
            dynamic>(websql, new { },
            _config.GetConnectionString("avecomfi_db"));
        if (OrderListFiltered == null)
        {
            OrderListFiltered = OrderList;
            OrderListFiltered.Reverse();
        }
    }
    private void SearchTermChanged(ChangeEventArgs changeEventArgs, string columTitle)
    {

        string searchText = changeEventArgs.Value.ToString();
        Console.WriteLine(searchText);

        if (columTitle == "Ordered Items")
        {
            OrderListFiltered = OrderList.Where(p => p.ItemsOrdered.Contains
            (searchText, StringComparison.OrdinalIgnoreCase) && p.PurchaseStatus.Contains("Comple")).ToList();
        }
        if (columTitle == "Status")
        {
            OrderListFiltered = OrderList.Where(p => p.PurchaseStatus.Contains
            (searchText, StringComparison.OrdinalIgnoreCase)).ToList();
        }
    }
    private void SearchDateChanged(ChangeEventArgs changeEventArgs, string columTitle)
    {

        string searchText = changeEventArgs.Value.ToString();
        Console.WriteLine(searchText);
        if (columTitle == "Date")
        {

            OrderListFiltered = OrderList.Where(p => p.PurchaseDate.ToString("yyyy-MM-dd").Contains
            (searchText) && p.PurchaseStatus.Contains("Comple")).ToList();
        }

    }
```

---
