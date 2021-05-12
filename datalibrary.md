# DataLibrary
|[DataLibrary](datalibrary.md)|[Blazor Setups](setups.md)|[UpdateApp](updateapp.md)|[Manual Update](manualupdate.md)|[Controls](controls.md)|[Order info](orderinfo.md)|[Models](models.md)|

I took the DataAccess library from Tim Corey and edited the LocalDataAccess library from it. I need both because the idea is to get web erp and local erp to communicate.
```csharp

using Dapper;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Threading.Tasks;

namespace DataLibrary
{
    public class DataAccess : IDataAccess
    {
        public async Task<List<T>> LoadWebData<T, U>(string websql, U parameters, string connectionString)
        {
            using (IDbConnection connection = new MySqlConnection(connectionString))
            {
                var rows = await connection.QueryAsync<T>(websql, parameters);

                return rows.ToList();
            }
        }

        public Task SaveWebData<T>(string websql, T parameters, string connectionString)
        {
            using (IDbConnection connection = new MySqlConnection(connectionString))
            {
                return connection.ExecuteAsync(websql, parameters);
            }
        }
    }

}


using System.Collections.Generic;
using System.Threading.Tasks;

namespace DataLibrary
{
    public interface IDataAccess
    {
        Task<List<T>> LoadWebData<T, U>(string sql, U parameters, string connectionString);
        Task SaveWebData<T>(string sql, T parameters, string connectionString);
    }

}


using Dapper;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Data.OleDb;
using System.Data;

namespace DataLibrary
{
    public class LocalDataAccess : ILocalDataAccess
    {
        public async Task<List<V>> LoadData<V, W>(string sql, W parameters, string connectionString)
        {
            using (IDbConnection connection = new OleDbConnection(connectionString))
            {
                var rows = await connection.QueryAsync<V>(sql, parameters);

                return rows.ToList();
            }
        }

        public Task SaveData<V>(string sql, V parameters, string connectionString)
        {
            using (IDbConnection connection = new OleDbConnection(connectionString))
            {
                return connection.ExecuteAsync(sql, parameters);
            }
        }
    }
}


using System.Collections.Generic;
using System.Threading.Tasks;
using System.Data.OleDb;
using System.Data;

namespace DataLibrary
{
    public interface ILocalDataAccess
    {
        Task<List<V>> LoadData<V, W>(string sql, W parameters, string connectionString);
        Task SaveData<V>(string sql, V parameters, string connectionString);

    }
}

```
