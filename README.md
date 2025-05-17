# php-login-system
 <!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Профили пользователей</title>
    <style>
        body {
            background: #f0f2f5;
            font-family: 'Verdana', sans-serif;
            margin: 0;
            padding: 20px;
        }
        .wrapper {
            max-width: 900px;
            margin: auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .form-block, .users-block {
            margin-bottom: 40px;
        }
        .user-card {
            border: 1px solid #ccc;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            background: #fafafa;
        }
        .user-info {
            flex: 1;
        }
        .user-info img {
            max-width: 100px;
            height: auto;
            border-radius: 8px;
            margin-top: 10px;
        }
        .form-block input[type="text"], .form-block textarea {
            width: 100%;
            padding: 8px;
            margin-top: 8px;
            margin-bottom: 16px;
            border: 1px solid #999;
            border-radius: 5px;
        }
        .form-block input[type="file"] {
            margin-bottom: 16px;
        }
        .form-block input[type="submit"], .delete-btn, .edit-btn {
            background-color: #4caf50;
            border: none;
            color: white;
            padding: 10px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 16px;
            text-decoration: none;
            display: inline-block;
            margin-top: 10px;
        }
        .delete-btn {
            background-color: #f44336;
        }
        .edit-btn {
            background-color: #2196F3;
        }
        .form-block input[type="submit"]:hover, .delete-btn:hover, .edit-btn:hover {
            opacity: 0.9;
        }
    </style>
</head>
<body>

<div class="wrapper">

    <div class="form-block">
        <h2>Добавить нового пользователя</h2>
        <form action="" method="post" enctype="multipart/form-data">
            <label>Имя:</label>
            <input type="text" name="name" required>

            <label>Возраст:</label>
            <input type="text" name="age" required>

            <label>О себе:</label>
            <textarea name="about" rows="4" required></textarea>

            <label>Фото:</label>
            <input type="file" name="photo">

            <input type="submit" name="add_user" value="Сохранить">
        </form>
    </div>

    <div class="users-block">
        <h2>Список пользователей</h2>
        <?php
        $uploadFolder = "uploads/";
        if (!file_exists($uploadFolder)) {
            mkdir($uploadFolder, 0777, true);
        }

        $dataFile = 'users.json';
        $users = file_exists($dataFile) ? json_decode(file_get_contents($dataFile), true) : [];

        
        if (isset($_GET['delete'])) {
            $deleteIndex = (int) $_GET['delete'];
            if (isset($users[$deleteIndex])) {
                if (!empty($users[$deleteIndex]['photo']) && file_exists($users[$deleteIndex]['photo'])) {
                    unlink($users[$deleteIndex]['photo']);
                }
                array_splice($users, $deleteIndex, 1);
                file_put_contents($dataFile, json_encode($users, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
                header("Location: " . strtok($_SERVER["REQUEST_URI"], '?'));
                exit;
            }
        }
        
        if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['add_user'])) {
            $name = htmlspecialchars($_POST['name']);
            $age = htmlspecialchars($_POST['age']);
            $about = htmlspecialchars($_POST['about']);
            $photoPath = '';

            if (!empty($_FILES['photo']['name'])) {
                $photoName = time() . '_' . basename($_FILES['photo']['name']);
                $targetPath = $uploadFolder . $photoName;
                if (move_uploaded_file($_FILES['photo']['tmp_name'], $targetPath)) {
                    $photoPath = $targetPath;
                }
            }

            $users[] = [
                'name' => $name,
                'age' => $age,
                'about' => $about,
                'photo' => $photoPath
            ];

            file_put_contents($dataFile, json_encode($users, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
            header("Location: " . $_SERVER['PHP_SELF']);
            exit;
        }

        
        if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['update_user'])) {
            $index = (int)$_POST['index'];
            if (isset($users[$index])) {
                $users[$index]['name'] = htmlspecialchars($_POST['name']);
                $users[$index]['age'] = htmlspecialchars($_POST['age']);
                $users[$index]['about'] = htmlspecialchars($_POST['about']);

                if (!empty($_FILES['photo']['name'])) {
                    if (!empty($users[$index]['photo']) && file_exists($users[$index]['photo'])) {
                        unlink($users[$index]['photo']);
                    }
                    $photoName = time() . '_' . basename($_FILES['photo']['name']);
                    $targetPath = $uploadFolder . $photoName;
                    if (move_uploaded_file($_FILES['photo']['tmp_name'], $targetPath)) {
                        $users[$index]['photo'] = $targetPath;
                    }
                }

                file_put_contents($dataFile, json_encode($users, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
                header("Location: " . $_SERVER['PHP_SELF']);
                exit;
            }
        }

        
        $editIndex = isset($_GET['edit']) ? (int)$_GET['edit'] : -1;

        if (!empty($users)) {
            foreach ($users as $index => $user) {
                echo '<div class="user-card">';
                echo '<div class="user-info">';
                if ($index === $editIndex) {
                    echo '<form action="" method="post" enctype="multipart/form-data">';
                    echo '<input type="hidden" name="index" value="' . $index . '">';
                    echo '<label>Имя:</label><br>';
                    echo '<input type="text" name="name" value="' . htmlspecialchars($user['name']) . '" required><br>';
                    echo '<label>Возраст:</label><br>';
                    echo '<input type="text" name="age" value="' . htmlspecialchars($user['age']) . '" required><br>';
                    echo '<label>О себе:</label><br>';
                    echo '<textarea name="about" rows="4" required>' . htmlspecialchars($user['about']) . '</textarea><br>';
                    echo '<label>Новое фото:</label><br>';
                    echo '<input type="file" name="photo"><br>';
                    echo '<input type="submit" name="update_user" value="Сохранить изменения">';
                    echo '</form>';
                } else {
                    echo '<strong>Имя:</strong> ' . htmlspecialchars($user['name']) . '<br>';
                    echo '<strong>Возраст:</strong> ' . htmlspecialchars($user['age']) . '<br>';
                    echo '<strong>О себе:</strong> ' . htmlspecialchars($user['about']) . '<br>';
                    if (!empty($user['photo'])) {
                        echo '<img src="' . $user['photo'] . '" alt="Фото пользователя">';
                    }
                }
                echo '</div>';
                echo '<div>';
                echo '<a class="delete-btn" href="?delete=' . $index . '" onclick="return confirm(\'Удалить эту запись?\')">Удалить</a> ';
                if ($index !== $editIndex) {
                    echo '<a class="edit-btn" href="?edit=' . $index . '">Изменить</a>';
                }
                echo '</div>';
                echo '</div>';
            }
        } else {
            echo "<p>Пока нет пользователей.</p>";
        }
        ?>
    </div>

</div>

</body>
</html>
