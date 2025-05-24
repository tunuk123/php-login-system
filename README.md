<?php
$host = "localhost";
$user = "root";
$pass = "";
$db = "tunuk1_db";

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die("Ошибка подключения: " . $conn->connect_error);
}

$uploadDir = "uploads";
if (!is_dir($uploadDir)) {
    mkdir($uploadDir, 0755, true);
}

$edit_entry = null;


if (isset($_GET['delete_id'])) {
    $id = intval($_GET['delete_id']);
    $conn->query("DELETE FROM users_info WHERE id = $id");
    header("Location: index.php");
    exit();
}


if (isset($_GET['edit_id'])) {
    $id = intval($_GET['edit_id']);
    $res = $conn->query("SELECT * FROM users_info WHERE id = $id");
    if ($res->num_rows > 0) {
        $edit_entry = $res->fetch_assoc();
    }
}


if ($_SERVER["REQUEST_METHOD"] === "POST" && isset($_POST['update'])) {
    $id = intval($_POST['id']);
    $surname = htmlspecialchars(trim($_POST['surname']));
    $firstname = htmlspecialchars(trim($_POST['firstname']));
    $birth = htmlspecialchars(trim($_POST['birth']));
    $bio = htmlspecialchars(trim($_POST['bio']));
    $imgPath = $_POST['existing_photo'] ?? '';

    if (!empty($_FILES["avatar"]["name"])) {
        $ext = pathinfo($_FILES["avatar"]["name"], PATHINFO_EXTENSION);
        $uniqueName = "img_" . uniqid() . "." . $ext;
        $fullPath = "$uploadDir/$uniqueName";
        if (move_uploaded_file($_FILES["avatar"]["tmp_name"], $fullPath)) {
            $imgPath = $fullPath;
        }
    }

    $stmt = $conn->prepare("UPDATE users_info SET name=?, surname=?, age=?, about=?, photo=? WHERE id=?");
    $stmt->bind_param("sisssi", $firstname, $surname, $birth, $bio, $imgPath, $id);
    $stmt->execute();
    $stmt->close();
    header("Location: index.php");
    exit();
}


if ($_SERVER["REQUEST_METHOD"] === "POST" && isset($_POST['submit'])) {
    $surname = htmlspecialchars(trim($_POST['surname']));
    $firstname = htmlspecialchars(trim($_POST['firstname']));
    $birth = htmlspecialchars(trim($_POST['birth']));
    $bio = htmlspecialchars(trim($_POST['bio']));
    $imgPath = "";

    if (!empty($_FILES["avatar"]["name"])) {
        $ext = pathinfo($_FILES["avatar"]["name"], PATHINFO_EXTENSION);
        $uniqueName = "img_" . uniqid() . "." . $ext;
        $fullPath = "$uploadDir/$uniqueName";
        if (move_uploaded_file($_FILES["avatar"]["tmp_name"], $fullPath)) {
            $imgPath = $fullPath;
        }
    }

    $stmt = $conn->prepare("INSERT INTO users_info (name, surname, age, about, photo) VALUES (?, ?, ?, ?, ?)");
    $stmt->bind_param("ssiss", $firstname, $surname, $birth, $bio, $imgPath);
    $stmt->execute();
    $stmt->close();
}

$result = $conn->query("SELECT * FROM users_info ORDER BY id DESC");
$entries = $result->fetch_all(MYSQLI_ASSOC);
?>

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Анкета</title>
    <style>
        body {
            background: #0f0f0f;
            font-family: 'Segoe UI', sans-serif;
            color: #fff;
            margin: 0;
            padding: 0;
        }
        .container {
            max-width: 720px;
            margin: 30px auto;
            padding: 20px;
            background: #1a1a1a;
            border-radius: 10px;
            box-shadow: 0 0 20px #0ff;
        }
        input, textarea {
            width: 100%;
            margin: 10px 0;
            padding: 10px;
            background: #222;
            border: 1px solid #0ff;
            color: #fff;
            border-radius: 4px;
        }
        input[type="submit"] {
            background: #00eaff;
            border: none;
            color: #000;
            font-weight: bold;
            cursor: pointer;
        }
        .btn {
            display: inline-block;
            padding: 6px 12px;
            margin: 5px 0;
            color: white;
            text-decoration: none;
            background: #00eaff;
            border-radius: 4px;
        }
        .btn.red {
            background: #ff0040;
        }
        .entries-container {
            margin-top: 30px;
        }
        .entry {
            display: flex;
            background: #111;
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 8px;
            box-shadow: 0 0 10px #0ff;
        }
        .entry img {
            max-width: 100px;
            max-height: 100px;
            border-radius: 6px;
            margin-right: 15px;
            object-fit: cover;
        }
        .entry .text {
            flex: 1;
        }
    </style>
</head>
<body>
<div class="container">
    <form action="" method="post" enctype="multipart/form-data">
        <h2><?= $edit_entry ? "Редактировать анкету" : "Новая анкета" ?></h2>
        <input type="hidden" name="id" value="<?= $edit_entry['id'] ?? '' ?>">
        <input type="hidden" name="existing_photo" value="<?= $edit_entry['photo'] ?? '' ?>">
        <input type="text" name="surname" placeholder="Фамилия" required value="<?= $edit_entry['surname'] ?? '' ?>">
        <input type="text" name="firstname" placeholder="Имя" required value="<?= $edit_entry['name'] ?? '' ?>">
        <input type="text" name="birth" placeholder="Дата рождения" required value="<?= $edit_entry['age'] ?? '' ?>">
        <input type="file" name="avatar" accept="image/*">
        <textarea name="bio" rows="3" placeholder="О себе" required><?= $edit_entry['about'] ?? '' ?></textarea>
        <input type="submit" name="<?= $edit_entry ? 'update' : 'submit' ?>" value="<?= $edit_entry ? 'Обновить' : 'Сохранить' ?>">
    </form>

    <div class="entries-container">
        <?php foreach ($entries as $entry): ?>
            <div class="entry">
                <?php if (!empty($entry['photo'])): ?>
                    <img src="<?= htmlspecialchars($entry['photo']) ?>" alt="Фото">
                <?php endif; ?>
                <div class="text">
                    <p><strong><?= htmlspecialchars($entry['name']) . ' ' . htmlspecialchars($entry['surname']) ?></strong></p>
                    <p>Дата рождения: <?= htmlspecialchars($entry['age']) ?></p>
                    <p><?= htmlspecialchars($entry['about']) ?></p>
                    <a href="?edit_id=<?= $entry['id'] ?>" class="btn">✏️ Редактировать</a>
                    <a href="?delete_id=<?= $entry['id'] ?>" class="btn red" onclick="return confirm('Удалить запись?');">🗑️ Удалить</a>
                </div>
            </div>
        <?php endforeach; ?>
    </div>
</div>
</body>
</html>

