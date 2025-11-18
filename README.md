 <!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>タイトル</title>
</head>
<body>
    <?php
    // データベース接続
    $dsn = 'mysql:dbname=tb270594db;host=localhost';
    $user = 'tb-270594';
    $password = 'w6fETMAwuw';
    $pdo = new PDO($dsn, $user, $password, array(PDO::ATTR_ERRMODE => PDO::ERRMODE_WARNING));

    // users情報　テーブル
    $sql = "CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )";
    $stmt = $pdo->exec($sql);

    // massages記録　テーブル
    $sql = "CREATE TABLE IF NOT EXISTS messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sender_id INT NOT NULL,
    receiver_id INT NOT NULL,
    content TEXT NOT NULL,
    sent_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    is_read BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (receiver_id) REFERENCES users(id)
    )";
    $stmt = $pdo->exec($sql);
    ?>
</body>
</html>
