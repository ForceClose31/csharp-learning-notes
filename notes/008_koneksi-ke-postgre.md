# Koneksi ke PostgreSQL

#### Daftar Isi

1. [Menginstall package Npgsql](#meningstall-package-npgsql)
1. [Membuat koneksi ke PostgreSQL](#membuat-koneksi-ke-postgresql)
1. [Membuat tabel](#membuat-tabel)
1. [Insert data](#insert-data)
1. [Select data](#select-data)

#### Meningstall package Npgsql

Langkah pertama sebelum dapat menghubungkan aplikasi C# dengan db PostgreSQL yaitu menginstall package **Npgsql**. Package tersebut digunakan sebagai penghubung koneksi ke PostgreSQL. Pastikan terhubung ke internet apabila hendak menginstall package.

![](./assets/Screenshot%202022-10-07%20060544.jpg)
Step 1: klik Project > Manage Nuget Packages

![](./assets/Screenshot%202022-10-07%20060814.jpg)
Step 2: Akan terbuka tab baru yaitu **NuGet: NamaProject**. Klik Browse, lalu cari Npgsql. Klik package Npgsql yang tampil.

![](./assets/Screenshot%202022-10-07%20061038.jpg)
Step 3: Klik Install, apabila muncul pemberitahuan cukup klik OK

Step 4: Setelah proses Install sudah selesai, tab NuGet dapat ditutup.

#### Membuat koneksi ke PostgreSQL

```cs
using Npgsql;

class Program
{
  static void Main(string[] args)
  {
    string connectionString = "Host=localhost;Username=postgres;Password=postgres;Database=fasilkom";
    NpgsqlConnection connection = new NpgsqlConnection(connectionString);
    connection.Open();
    // sekumpulan perintah SQL
    connection.Close();
  }
}
```

- Langkah pertama mengimport package Npgsql yang sebelumnya sudah diinstall menggunakan `using Npgsql`.
- Selanjutnya membuat sebuah string yang menjadi konfigurasi untuk koneksi ke database. Konfigurasi berisi data **key=value** yang dipisahkan dengan titik koma **;**.
  - `Host`, Hostname/letak PostgreSQL berjalan
  - `Port`, Port PostgreSQL berjalan [default: 5432]
  - `Username`, username user PostgreSQL
  - `Password`, password user PostgreSQL
  - `Database`, database PostgreSQL
    > Pastikan database sudah ada sebelum melakukan koneksi
- Membuat koneksi ke PostgreSQL dengan cara membuat objek dari class `NpgsqlConnection` yang menerima satu parameter yaitu string konfigurasi.
- Memanggil method `.Open()` untuk membuka koneksi
- Pastikan memanggil method `.Close()` apabila koneksi sudah tidak digunakan lagi.

Kode diatas telah membuat koneksi ke PostgreSQL, namun belum ada perintah SQL yang dijalankan. Untuk membuat perintah perlu membuat objek dari class `NpgsqlCommand`

#### Membuat tabel

Sebelum membuat tabel, pastikan tidak ada tabel yang duplikat nantinya. yaitu dengan menghapus tabel apabila tabel sudah ada di databse dengan cara:

```cs
// cara 1
string cmdText = "DROP TABLE IF EXISTS mahasiswa";
NpgsqlCommand command = new NpgsqlCommand(cmdText, connection);
command.ExecuteNonQuery();
```

```cs
// cara 2
NpgsqlCommand command = new NpgsqlCommand();
command.Connection = connection;
command.CommandText = "DROP TABLE IF EXISTS mahasiswa";
command.ExecuteNonQuery();
```

- `NpgsqlCommand` memerlukan 2 properti sebelum dijalankan. yaitu perintahnya, kita sebut dengan `CommandText`. dan koneksi ke PostgreSQL nya, kita sebut dengan `Connection`
- Memanggil method `.ExecuteNonQuery()` karena perintah tersebut tidak mereturn data apapun.

Setelah menghindari adanya duplikat tabel, langka selanjutnya membuat tabel.

```cs
command.CommandText = "CREATE TABLE mahasiswa (id SERIAL PRIMARY KEY, nama VARCHAR)";
command.ExecuteNonQuery();
```

Dalam penggunaan command diatas tidak dibungkus menggunakan using, dalam best practicenya penulisan query lebih baik dibungkus ke dalam using supaya tidak terjadi kebocoran sumber daya

```cs
string insertQuery = "CREATE TABLE mahasiswa (id SERIAL PRIMARY KEY, nama VARCHAR)";
using (var cmd = new NpgsqlCommand(insertQuery, conn))
{
    cmd.ExecuteNonQuery();
}
```

#### Insert data

Untuk menginsert data disarankan value data tidak dituliskan secara langsung seperti berikut.

`INSERT INTO mahasiswa(nim, nama) VALUES(202410101000, 'Budi');`

Melainkan dengan menggunakan Prepared Statement, agar insert data lebih **aman**. yaitu dengan cara berikut.

```cs
command.CommandText = "INSERT INTO mahasiswa(nama) VALUES(@nama)";

command.Parameters.AddWithValue("nama", "Budi");
command.Prepare();
command.ExecuteNonQuery();
command.Parameters.Clear();

command.Parameters.AddWithValue("nama", "Andi");
command.Prepare();
command.ExecuteNonQuery();
command.Parameters.Clear();
```

- values ditandai dengan awalan `@` diikuti dengan nama value (bebas).
- Selanjutnya parameter diisi dengan memanggil method pada NpgsqlCommand, yaitu pada `.Parameters.AddWithValue()`.
- Parameter pertama merupakan nama yang sudah kita tuliskan sebelumnya pada perintah SQL, `@nama -> nama`. Sedangkan parameter kedua berisi nilai yang hendak dimasukkan.
- Selanjutnya memanggil method `.Prepare()` dan method `.ExecuteNonQuery()`.
- Setelah perintah dijalankan, disarankan memanggil method `.Parameters.Clear()` agar insert data selanjutnya dapat memiliki value yang berbeda.

```cs
```cs
string insertQuery = "INSERT INTO mahasiswa (name) VALUES (@name)";
using (var cmd = new NpgsqlCommand(insertQuery, conn))
{
    cmd.Parameters.AddWithValue("name", "Bashori");
    int rowsAffected = cmd.ExecuteNonQuery();
    Console.WriteLine($"{rowsAffected} row(s) inserted.");
}
```
```

#### Select data

Untuk melakukan query `SELECT`, perintah dimasukkan pada CommandText seperti sebelumnya.

```cs
command.CommandText = "SELECT * FROM mahasiswa";
```

Namun method yang dipanggil bukan .ExecuteNonQuery() melainkan `.ExecuteReader()`. Method tersebut akan mengembalikan sebuah objek dari class `NpgsqlDataReader`

```cs
command.CommandText = "SELECT * FROM mahasiswa";
NpgsqlDataReader reader;
reader = command.ExecuteReader();
```

Cara kedua 

```cs
string selectQuery = "SELECT * FROM mahasiswa";
using (var cmd = new NpgsqlCommand(selectQuery, conn))
{
    using (var reader = cmd.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine($"ID: {reader["id"]}, Name: {reader["name"]}");
        }
    }
}

```

Pembacaan hasil query dilakukan per baris dengan memanggil method `.Read()`.

```cs
reader.Read(); // masuk baris ke-1
reader.Read(); // lanjut baris ke-2
```

daripada menuliskan method .Read() secara berulang, lebih baik menggunakan looping sembari memanggil method tersebut. Apabila baris sudah habis maka looping akan berhenti.

```cs
while (reader.Read()) {
  // iterasi setiap baris
}
```

Untuk mengambil datanya menggunakan method sesuai dengan tipe data pada database, dan parameter dari method tersebut merupakan indeks kolom (misalnya ada kolom id dan nama, maka indeks id=0 dan nama=1). Misalnya id bertipe data SERIAL maka perlu memaggil method `.GetInt32(index)` dan nama bertipe data VARCHAR maka perlu memanggil method `.GetString(index)`

```cs
while (reader.Read()) {
  int id = reader.GetInt32(0);
  string nama = reader.GetString(1);
  Console.WriteLine($"{id} {nama}");
}
```

| method     | return type |
| ---------- | ----------- |
| GetByte    | byte        |
| GetChar    | char        |
| GetDecimal | decimal     |
| GetDouble  | double      |
| GetFloat   | float       |
| GetInt16   | short       |
| GetInt32   | int         |
| GetInt64   | long        |

[Lihat lebih lengkap](https://www.npgsql.org/doc/api/Npgsql.NpgsqlDataReader.html#methods)

#### Update Data

```cs
string updateQuery = "UPDATE mahasiswa SET name = @name WHERE id = @id";
using (var cmd = new NpgsqlCommand(updateQuery, conn))
{
    cmd.Parameters.AddWithValue("name", "Ady Ganss");
    cmd.Parameters.AddWithValue("id", 1);
    int rowsAffected = cmd.ExecuteNonQuery();
    Console.WriteLine($"{rowsAffected} row(s) updated.");
}
```

#### Delete Data

```cs
string deleteQuery = "DELETE FROM mahasiswa WHERE id = @id";
using (var cmd = new NpgsqlCommand(deleteQuery, conn))
{
    cmd.Parameters.AddWithValue("id", 1);
    int rowsAffected = cmd.ExecuteNonQuery();
    Console.WriteLine($"{rowsAffected} row(s) deleted.");
}
```

---

#### Contoh penggunaan

```cs
using Npgsql;

class Program
{
    static string connString = "Host=localhost;Username=postgres;Password=QWEASDZXC31;Database=sampledb";

    static void Main(string[] args)
    {
        using (var conn = new NpgsqlConnection(connString))
        {
            try
            {
                conn.Open();
                Console.WriteLine("Koneksi ke database berhasil!");
                CreateTable(conn);

                int choice;
                do
                {
                    Console.WriteLine("\nCRUD Operations Menu");
                    Console.WriteLine("1. Add User");
                    Console.WriteLine("2. View Users");
                    Console.WriteLine("3. Update User");
                    Console.WriteLine("4. Delete User");
                    Console.WriteLine("5. Exit");
                    Console.Write("Choose an option: ");
                    choice = Convert.ToInt32(Console.ReadLine());
                    switch (choice)
                    {
                        case 1:
                            AddUser(conn);
                            break;
                        case 2:
                            ViewUsers(conn);
                            break;
                        case 3:
                            UpdateUser(conn);
                            break;
                        case 4:
                            DeleteUser(conn);
                            break;
                    }
                } while (choice != 5);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Terjadi kesalahan: {ex.Message}");
            }
        }
    }

    static void CreateTable(NpgsqlConnection conn)
    {
        string createTableQuery = @"
                DROP TABLE IF EXISTS users;
                CREATE TABLE users (
                    id SERIAL PRIMARY KEY,
                    name VARCHAR(100) NOT NULL,
                    email VARCHAR(100) NOT NULL,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                );
            ";

        using (var cmd = new NpgsqlCommand(createTableQuery, conn))
        {
            cmd.ExecuteNonQuery();
            Console.WriteLine("Tabel 'users' berhasil dibuat ulang.");
        }
    }

    static void AddUser(NpgsqlConnection conn)
    {
        Console.Write("Enter Name: ");
        string? name = Console.ReadLine();
        Console.Write("Enter Email: ");
        string? email = Console.ReadLine();

        name ??= "Default";
        email ??= "Default";
        string insertQuery = "INSERT INTO users (name, email) VALUES (@name, @email)";
        using (var cmd = new NpgsqlCommand(insertQuery, conn))
        {
            cmd.Parameters.AddWithValue("name", name);
            cmd.Parameters.AddWithValue("email", email);
            cmd.ExecuteNonQuery();
            Console.WriteLine("User berhasil ditambahkan!");
        }
    }

    static void ViewUsers(NpgsqlConnection conn)
    {
        string selectQuery = "SELECT * FROM users";
        using (var cmd = new NpgsqlCommand(selectQuery, conn))
        {
            using (var reader = cmd.ExecuteReader())
            {
                Console.WriteLine("\nDaftar Pengguna:");
                while (reader.Read())
                {
                    Console.WriteLine($"ID: {reader["id"]}, Name: {reader["name"]}, Email: {reader["email"]}, Created At: {reader["created_at"]}");
                }
            }
        }
    }

    static void UpdateUser(NpgsqlConnection conn)
    {
        Console.Write("Enter User ID to Update: ");
        int id = Convert.ToInt32(Console.ReadLine());
        Console.Write("Enter New Name: ");
        string? name = Console.ReadLine();

        name ??= "Default";

        string updateQuery = "UPDATE users SET name = @name WHERE id = @id";
        using (var cmd = new NpgsqlCommand(updateQuery, conn))
        {
            cmd.Parameters.AddWithValue("name", name);
            cmd.Parameters.AddWithValue("id", id);
            int rowsAffected = cmd.ExecuteNonQuery();
            if (rowsAffected > 0)
                Console.WriteLine("User berhasil diperbarui!");
            else
                Console.WriteLine("User tidak ditemukan.");
        }
    }

    static void DeleteUser(NpgsqlConnection conn)
    {
        Console.Write("Enter User ID to Delete: ");
        int id = Convert.ToInt32(Console.ReadLine());

        string deleteQuery = "DELETE FROM users WHERE id = @id";
        using (var cmd = new NpgsqlCommand(deleteQuery, conn))
        {
            cmd.Parameters.AddWithValue("id", id);
            int rowsAffected = cmd.ExecuteNonQuery();
            if (rowsAffected > 0)
                Console.WriteLine("User berhasil dihapus!");
            else
                Console.WriteLine("User tidak ditemukan.");
        }
    }
}
```
