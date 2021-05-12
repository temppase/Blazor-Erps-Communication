|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|

---

# Models

Something about models...

## LocalDbModel
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace AvecomBlazorApp.Models
{
    public class LocalDbModel
    {
        public int Id { get; set; }
        public string Sku { get; set; }
        public string Valmistaja { get; set; }
        public string Tuote { get; set; }
        public int Hinta { get; set; }
        public int Tarjoushinta { get; set; }
        public int Ostohinta { get; set; }
        public string Saldo { get; set; }
        public string Luokat { get; set; }
        public int Kg { get; set; }
        public bool Webtuote { get; set; }
        public string Tuotekuvaus { get; set; }
        public string Tuotetiedot { get; set; }
    }
}
```
## WebDbModel
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace AvecomBlazorApp.Models
{
    public class WebDbModel
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public string SKU { get; set; }
        public string Stock { get; set; }
    }
}
```
## OrderInfoModel
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace AvecomBlazorApp.Models
{
    public class OrderInfoModel
    {
        public int OrderID { get; set; }
        public DateTime PurchaseDate { get; set; }
        public string ItemsOrdered { get; set; }
        public string PurchaseStatus { get; set; }
    }
}
```

---
