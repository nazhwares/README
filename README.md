# README
# Modul Praktikum: Web Service REST untuk Manajemen Buku

## Tujuan
Membuat dan menguji web service REST untuk manajemen buku menggunakan PHP dan MySQL.

## Alat yang Dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)
3. Postman

## Langkah-langkah Praktikum

### 1. Persiapan Lingkungan
1. Instal XAMPP jika belum ada.
2. Buat folder baru bernama `cakes` di dalam direktori `htdocs` XAMPP Anda.

### 2. Membuat Database
1. Buka phpMyAdmin (http://localhost/phpmyadmin)
2. Buat database baru bernama `cakes`
3. Pilih database `cakes`, lalu buka tab SQL
4. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:

```sql
CREATE TABLE books (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
    flavor VARCHAR(255) NOT NULL,
    price VARCHAR(255) NOT NULL,
    stock INT NOT NULL
);

INSERT INTO books (name,flavor,price,stock) VALUES
('donat','coklat',15000,15),
('roti keset','keju,7000,9),
('roti O','mocca',5000,15),
('roti klasik','coklat keju',7500,15),
('cromboloni','strawbery,15000,11);
```

### 3. Membuat File PHP untuk Web Service
1. Buka text editor Anda.
2. Buat file baru dan simpan sebagai `cakes.php` di dalam folder `cakes`.
3. Salin dan tempel kode berikut ke dalam `cakes.php`:

```php
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, flavorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'],'/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'cakes';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM cakes WHERE id = ?");
            $stmt->execute([$id]);
            $cakes = $stmt->fetch();
            if ($cakes) {
                response(200, $cakes);
            } else {
                response(404, ["message" => "cakes not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM cakes");
            $cakes = $stmt->fetchAll();
            response(200, $cakes);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->flavor) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO cakes (name, flavor, price,stock) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->flavor, $data->price, $data->stock])) {
            response(201, ["message" => "cakes created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create cakes"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "cakes ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->flavor) || !isset($data->price)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE cakes SET name = ?, flavor = ?, price = ?, stock = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->flavor, $data->price, $data->stock, $id])) {
            response(200, ["message" => "cakes updated"]);
        } else {
            response(500, ["message" => "Failed to update cakes"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "cakes ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM cakes WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "cakes deleted"]);
        } else {
            response(500, ["message" => "Failed to delete cakes"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
```

### 4. Pengujian dengan Postman
1. Buka Postman
2. Buat request baru untuk setiap operasi berikut:

#### a. GET All Books
- Method: GET
- URL: `http://localhost/cakes/cakes.php`
- Klik "Send"

#### b. GET Specific Book
- Method: GET
- URL: `http://localhost/cakes/cakes.php/1` (untuk buku dengan ID 1)
- Klik "Send"

#### c. POST New Book
- Method: POST
- URL: `http://localhost/cakes/cakes.php`
- Headers: 
  - Key: Content-Type
  - Value: application/json
- Body:
  - Pilih "raw" dan "JSON"
  - Masukkan:
    ```json
    {
        "name": "cromboloni",
        "flavor": "strawbery",
        "price": 15.000
    }
    ```
- Klik "Send"

#### d. PUT (Update) Book
- Method: PUT
- URL: `http://localhost/cakes/cakes.php/6` (asumsikan ID buku baru adalah 6)
- Headers: 
  - Key: Content-Type
  - Value: application/json
- Body:
  - Pilih "raw" dan "JSON"
  - Masukkan:
    ```json
    {
        "nama": "donat",
        "flavor": "coklat",
        "price": 15.000
    }
    ```
- Klik "Send"

#### e. DELETE Book
- Method: DELETE
- URL: `http://localhost/cakes/cakes.php/6` (untuk menghapus buku dengan ID 6)
- Klik "Send"

### Kesimpulan
Dalam praktikum ini, Anda telah berhasil membuat web service REST untuk manajemen buku menggunakan PHP dan MySQL. Anda juga telah belajar cara menguji API menggunakan Postman. Praktik ini memberikan dasar yang kuat untuk pengembangan API RESTful lebih lanjut.
### HASIL
<img width="919" alt="delete" src="https://github.com/user-attachments/assets/58a06601-b1cf-42fc-9ff3-ccc84ca441c2"> (delete)

<img width="918" alt="get nw " src="https://github.com/user-attachments/assets/f391190f-f904-4a6d-936b-36c89fefa4db"> (getnw)

<img width="928" alt="getnw 2" src="https://github.com/user-attachments/assets/a78fa7f7-dbc1-44a7-b15b-b9ecf9339fa0"> (getnw 2)

<img width="927" alt="post" src="https://github.com/user-attachments/assets/01c89d56-e056-4c08-bc2b-ccd6ad5b6469"> (post)

<img width="929" alt="put" src="https://github.com/user-attachments/assets/727dd703-d9d9-4216-a4cb-b46f53bfd820"> (put)
