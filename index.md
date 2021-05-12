## DataLibrary

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
## Semi Automatic update

So far, we haven’t ended up fully automating the update. Full automation can also bring problems when the program is currently applying for changes if something has been sold locally or on the web. The problem is that if the quantity of a product is increased in either, then automation would see it as a sale in another.
