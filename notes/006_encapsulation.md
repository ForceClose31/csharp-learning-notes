# Encapsulation

#### Daftar Isi

1. [Encapsulation](#encapsulation)
1. [Access Modifier](#access-modifier)
1. [Contoh Encapsulation](#encapsulation)

#### Encapsulation

Encapsulation adalah salah satu pilar utama dalam **Object-Oriented Programming (OOP)** yang bertujuan untuk menyembunyikan detail implementasi internal suatu objek dari pengguna luar. Dalam konsep ini, data (variabel) dan metode (fungsi) digabungkan dalam satu kesatuan yang disebut kelas dan akses ke data tersebut dikontrol melalui mekanisme **access modifier**.

Encapsulation memungkinkan data dalam objek dilindungi agar tidak diakses atau diubah sembarangan oleh objek lain di luar kelasnya. Ini memastikan bahwa data hanya dapat dimodifikasi melalui metode yang sesuai, menjaga integritas data.

#### Access Modifier

Berbicara mengenai encapsulation, _Access Modifier_ menjadi salah satu topik yang perlu dipelajari. Access modifier adalah kata kunci dalam C# yang digunakan untuk mengatur **tingkat aksesibilitas** anggota kelas (seperti variabel dan metode).
Access modifier terdiri dari:

- `public`, class member dapat diakses dari **manapun**
- `private`, class member hanya dapat diakses dari **class yang sama**
- `protected`, class member dapat diakses dari **class yang sama** dan **child class nya**
- `internal`, class member dapat diakses dari **manapun** selama berasal dari 1 assembly yg sama.
- `protected internal`, class member dapat diakses dari **manapun** selama masih 1 assembly atau dapat diakses dari assembly yg berbeda dengan syarat merupakan **child class** dari class yang bersangkutan
- `private protected`, class member dapat diakses dari **child class nya**

#### Contoh access modifier

```cs
using System;

namespace AccessModifierDemo
{
    public class Employee
    {
        // Variabel private hanya dapat diakses dari dalam kelas
        private int id;

        // Property public dapat diakses dari mana saja
        public string Name { get; set; }

        // Variabel protected hanya dapat diakses dari kelas ini atau subclass
        protected decimal salary;

        // Method internal hanya dapat diakses dari dalam assembly/proyek ini
        internal void SetID(int empID)
        {
            id = empID;
        }

        // Method protected internal dapat diakses dari kelas turunan atau dalam proyek yang sama
        protected internal void SetSalary(decimal empSalary)
        {
            salary = empSalary;
        }
    }

    // Class Manager mewarisi dari Employee
    public class Manager : Employee
    {
        public void DisplayManagerInfo()
        {
            // Dapat mengakses variabel protected dari kelas induk
            Console.WriteLine($"Manager Salary: {salary}");
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Employee emp = new Employee();
            Manager mgr = new Manager();

            // Akses property public
            emp.Name = "John";

            // Akses method internal dalam proyek yang sama
            emp.SetID(123);

            // Akses method protected internal dalam proyek yang sama
            mgr.SetSalary(5000);

            mgr.DisplayManagerInfo();
        }
    }
}
```
##### Penjelasan :

- Variabel `id` bersifat `private`, hanya dapat diakses dari dalam kelas `Employee`.
- Property `Name` bersifat `public`, sehingga bisa diakses dari mana saja.
- Variabel `salary` bersifat `protected`, jadi hanya dapat diakses dari kelas `Employee` atau kelas turunannya seperti `Manager`.
- Method `SetID` bersifat `internal`, sehingga hanya bisa diakses dari dalam proyek yang sama.
- Method `SetSalary` bersifat `protected internal`, dapat diakses baik dari kelas turunan maupun dari dalam proyek yang sama.

#### Contoh Encapsulation

Penerapan encapsulation pada property yaitu dengan cara membatasi hak read-write secara langsung, melainkan melalui method setter dan getter.

Contoh:

```cs
class Mahasiswa
{
  private string nama;

  public void SetNama(string nama)
  {
    this.nama = nama;
  }

  public string GetNama()
  {
    return this.nama;
  }
}
```

```cs
Mahasiswa mahasiswa = new Mahasiswa();
// mahasiswa.nama = "Budi"; // Error
// Console.WriteLine(mahasiswa.nama); // Error
mahasiswa.SetNama("Budi");
Console.WriteLine(mahasiswa.GetNama());
```

Pada bahasa pemrograman JAVA, setter dan getter dituliskan sama seperti kode diatas, yaitu dengan menambahkan method setter dan getter untuk mengakses private property. Sedangkan pada C# dapat menggunakan cara diatas, atau cara _best practices_ C# yang hanya mengubah perilaku ketika data diubah/dipanggil.

```cs
class Mahasiswa
{
  private string nama;
  public string Nama
  {
    get { return this.nama; }
    set { this.nama = value; }
  }
}
```

```cs
Mahasiswa mahasiswa = new Mahasiswa();
// mahasiswa.nama = "Budi"; // Error
// Console.WriteLine(mahasiswa.nama); // Error
mahasiswa.Nama = "Budi";
Console.WriteLine(mahasiswa.Nama);
```

<details>
<summary>Contoh penggunaan</summary>

```cs
class Persegi
{
  private int panjang;
  private int lebar;
  private int luas;

  public int Panjang
  {
    get { return this.panjang; }
    set
    {
      this.panjang = value;
      this.UpdateLuas();
    }
  }

  public int Lebar
  {
    get { return this.lebar; }
    set
    {
      this.lebar = value;
      this.UpdateLuas();
    }
  }

  public int Luas
  {
    get { return this.luas; }
  }

  private void UpdateLuas()
  {
    this.luas = this.panjang * this.lebar;
  }
}
```

```cs
Persegi persegi = new Persegi();
persegi.Lebar = 12;
persegi.Panjang = 20;
Console.WriteLine(persegi.Luas); // Output: 240
// persegi.Luas = 120; // Error, karena tidak memiliki fungsi setter
```

</details>
