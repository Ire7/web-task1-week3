# User Status Management Web Task

## Project Title
**Web Task 3 – PHP + MySQL User Status Management**

## Description
This project is a simple PHP-based web application to manage users by name and age, and toggle their status (active/inactive).  
It uses MySQL for database storage and runs on XAMPP (Apache + MySQL).

## Requirements
XAMPP (Apache, MySQL)
PHP 7.x or later
MySQL
A web browser (Chrome, Edge, etc.)
Visual Studio Code (for editing)

## Setup Steps

### 1. Create the Database and Table
 Open phpMyAdmin via:  
  http://localhost/phpmyadmin
 Run the following SQL code:

sql
CREATE DATABASE IF NOT EXISTS user_status_db;

USE user_status_db;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    age INT,
    status TINYINT(1) DEFAULT 0
);

### 2. Add the PHP File
Go to:  
  C:\xampp\htdocs\
Create a new folder named:  
  web_task3
Inside it, create a file named `index.php`, and paste the following code:

php
<?php
$conn = new mysqli("localhost", "root", "", "user_status_db");
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

if (isset($_POST['submit'])) {
    $name = $_POST['name'];
    $age = $_POST['age'];

    if (!empty($name) && !empty($age)) {
        $stmt = $conn->prepare("INSERT INTO users (name, age) VALUES (?, ?)");
        $stmt->bind_param("si", $name, $age);
        $stmt->execute();
        $stmt->close();
    }
}

if (isset($_GET['toggle'])) {
    $id = $_GET['toggle'];
    $result = $conn->query("SELECT status FROM users WHERE id = $id");
    if ($row = $result->fetch_assoc()) {
        $newStatus = $row['status'] == 1 ? 0 : 1;
        $conn->query("UPDATE users SET status = $newStatus WHERE id = $id");
    }
}

$users = $conn->query("SELECT * FROM users");
?>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Users List</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid black; padding: 10px; text-align: center; }
        th { background-color: #ddd; }
    </style>
</head>
<body>

<form method="POST" action="">
    Name: <input type="text" name="name">
    Age: <input type="number" name="age">
    <input type="submit" name="submit" value="Submit">
</form>

<br><br>

<table>
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Age</th>
        <th>Status</th>
        <th>Action</th>
    </tr>
    <?php
    if ($users->num_rows > 0) {
        while ($row = $users->fetch_assoc()) {
            echo "<tr>
                <td>{$row['id']}</td>
                <td>{$row['name']}</td>
                <td>{$row['age']}</td>
                <td>{$row['status']}</td>
                <td><a href='?toggle={$row['id']}'>Toggle</a></td>
            </tr>";
        }
    }
    ?>
</table>

</body>
</html>
-----------------------------------------------
## How to Use the Project?

1. Start **Apache** and **MySQL** from XAMPP Control Panel.
2. Open your browser and go to:  
   http://localhost/web_task3/index.php
3. Enter a name and age, then click **Submit**.
4. The user will appear in the table.
5. Click on **Toggle** to change the user's status (0 ↔ 1).

## Notes
 The database name must be `user_status_db`.
 The table must be named `users`.
 All fields must be properly filled to insert a new user.
 The `status` field defaults to 0 and toggles when clicking the action link.
 If default users appear (like "Michael"), make sure to clear your table in phpMyAdmin.


## Known Issues & Solutions

### 1. IDs Generated Randomly or Skipped

**Issue:**  
We noticed that the user `ID` values were sometimes not sequential (e.g., 1, 2, 5, 6...), or that entries with lower IDs kept showing up again and again after deletion.

**Cause:**  
This happened because:
The ID field is set to AUTO_INCREMENT, so if a user is deleted, the ID is not reused.
Some test entries were inserted manually or reloaded by mistake.
The form re-submitted automatically when refreshing the page, causing duplicate data or strange IDs.

**Solution:**  
Clear the table in phpMyAdmin using:  
  TRUNCATE TABLE users;  
  This resets the IDs back to 1.
Avoid refreshing the page after submitting — or use `header("Location: index.php");` after insertion to prevent form resubmission (optional improvement).

### 2. Status Field Always Set to 0

**Issue:**  
All new users were added with status = 0.

**Cause:**  
This is because the `status` field has a default value of 0 in the database schema, and no value is provided during user insertion.

**Solution:**  
This behavior is intentional for simplicity. You can toggle the status manually using the toggle link in the table. If needed, a status toggle during insertion can be implemented with logic in PHP.

## this is the code 

<?php
// الاتصال بقاعدة البيانات
$conn = new mysqli("localhost", "root", "", "user_status_db");

// التحقق من الاتصال
if ($conn->connect_error) {
    die("فشل الاتصال: " . $conn->connect_error);
}

// إذا تم الضغط على زر الإرسال
if (isset($_POST['submit'])) {
    $name = $_POST['name'];
    $age = $_POST['age'];

    if (!empty($name) && !empty($age)) {
    $stmt = $conn->prepare("INSERT INTO users (name, age) VALUES (?, ?)");
    $stmt->bind_param("si", $name, $age);
    $stmt->execute();
    $stmt->close();

    //حل إعادة الإرسال التلقائي
    header("Location: " . $_SERVER['PHP_SELF']);
    exit;
}

}

//إذا تم طلب التبديل (toggle) للحالة
if (isset($_GET['toggle'])) {
    $id = $_GET['toggle'];

    // نجيب الحالة الحالية
    $result = $conn->query("SELECT status FROM users WHERE id = $id");
    if ($row = $result->fetch_assoc()) {
        $newStatus = $row['status'] == 1 ? 0 : 1;
        $conn->query("UPDATE users SET status = $newStatus WHERE id = $id");
    }
}

// عرض المستخدمين
$users = $conn->query("SELECT * FROM users");
?>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Users List</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid black; padding: 10px; text-align: center; }
        th { background-color: #ddd; }
    </style>
</head>
<body>

<form method="POST" action="">
    Name: <input type="text" name="name">
    Age: <input type="number" name="age">
    <input type="submit" name="submit" value="Submit">
</form>

<br><br>

<table>
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Age</th>
        <th>Status</th>
        <th>Action</th>
    </tr>
    <?php
    if ($users->num_rows > 0) {
        while ($row = $users->fetch_assoc()) {
            echo "<tr>
                <td>{$row['id']}</td>
                <td>{$row['name']}</td>
                <td>{$row['age']}</td>
                <td>{$row['status']}</td>
                <td><a href='?toggle={$row['id']}'>Toggle</a></td>
            </tr>";
        }
    }
    ?>
</table>

</body>
</html>
