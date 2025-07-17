# web-task1-week3
design a webpage 
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
