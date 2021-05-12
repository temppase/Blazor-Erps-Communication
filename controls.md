# Controls
|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|
The controllers are used, for example, for manual updating and retrieval of order information.

## CustomTable Component
```html
@typeparam TItem
<div class="table-wrapper-scroll-y my-custom-scrollbar">
    <table class="table table-bordered table-striped mb-0">
        <thead class="bg-dark text-light sticky-top">
            @TableHeader
        </thead>
        <tbody>
            
            @foreach (var item in Items)
            {

                <tr id="@i">
                    @RowTemplate(item)
                </tr>
                i++;
            }
        </tbody>
    </table>
</div>
    
```
### Code section
```csharp
    [Parameter]
    public RenderFragment TableHeader { get; set; }
    [Parameter]
    public RenderFragment<TItem> RowTemplate { get; set; }
    [Parameter]
    public IReadOnlyList<TItem> Items { get; set; }
    [Parameter]
    public int i { get; set; }
```
Very simple custom table...

## GridColum Component
```html
<th>
    <div class="col-10 row">
        @ColumTitle
        <input class="form-control" placeholder="search" @oninput="OnSearchTextChange" value="@Value"/>
    </div>
</th>
```
### Code section
```csharp
[Parameter]
public string ColumTitle { get; set; }
[CascadingParameter(Name = "ManualUpdateTable")]
public string Value { get; set; }
[Parameter]
public EventCallback<ChangeEventArgs> OnSearchTextChange { get; set; }
```
The GridColum component is used in the table header search function...
## GridColumCalendar
This is used for date search...
```html
<th>
    <div class="col-10 row">
        @ColumTitle
        <input type="date"class="form-control" placeholder="date" @oninput="OnSearchDateChange" value="@Value" />
    </div>
</th>
```
### Code section
```csharp
    [Parameter]
    public string ColumTitle { get; set; }
    [CascadingParameter(Name = "ManualUpdateTable")]
    public DateTime Value { get; set; }
    [Parameter]
    public EventCallback<ChangeEventArgs> OnSearchDateChange { get; set; }
```
