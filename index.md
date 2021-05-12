## DataLibrary

```csharp

// Web connection

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

// Interface

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

// Local connection

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

// Interface

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

´´´

