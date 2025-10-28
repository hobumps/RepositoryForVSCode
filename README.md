<?php
session_start();

/* ====== DB接続 ====== */
$dsn = 'mysql:host=localhost;dbname=tb270594db;charset=utf8mb4';
$user = 'tb-270594';
$password = 'w6fETMAwuw';

try {
    $pdo = new PDO($dsn, $user, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]);
} catch (PDOException $e) {
    exit('DB接続エラー: ' . $e->getMessage());
}

/* ====== 初期設定 ====== */
function h($s){ return htmlspecialchars($s, ENT_QUOTES, 'UTF-8'); }

if (empty($_SESSION['token'])) {
    $_SESSION['token'] = bin2hex(random_bytes(32));
}

$errors = [];
$done = false;
$username = '';

/* ====== POST受信 ====== */
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!isset($_POST['token']) || !hash_equals($_SESSION['token'], $_POST['token'])) {
        $errors[] = '不正なリクエストです。';
    } else {
        $username = trim($_POST['username'] ?? '');
        $pw_plain = $_POST['password'] ?? '';
        $pw_conf  = $_POST['password_confirm'] ?? '';

        // 入力チェック
        if ($username === '') {
            $errors[] = 'ユーザー名を入力してください。';
        } elseif (!preg_match('/^[a-zA-Z0-9_]{3,20}$/', $username)) {
            $errors[] = 'ユーザー名は英数字と_で3〜20文字にしてください。';
        }

        if ($pw_plain === '') {
            $errors[] = 'パスワードを入力してください。';
        } elseif (strlen($pw_plain) < 8) {
            $errors[] = 'パスワードは8文字以上にしてください。';
        }

        if ($pw_plain !== $pw_conf) {
            $errors[] = '確認用パスワードが一致しません。';
        }

        // 既存ユーザー名チェック
        if (!$errors) {
            $stmt = $pdo->prepare('SELECT 1 FROM users WHERE username = ? LIMIT 1');
            $stmt->execute([$username]);
            if ($stmt->fetch()) {
                $errors[] = 'このユーザー名は既に登録されています。';
            }
        }

        // 登録処理
        if (!$errors) {
            $hash = password_hash($pw_plain, PASSWORD_DEFAULT);
            $stmt = $pdo->prepare('INSERT INTO users (username, password) VALUES (?, ?)');
            $stmt->execute([$username, $hash]);
            unset($_SESSION['token']);
            $done = true;
        }
    }
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>新規登録</title>
<style>
body {font-family: sans-serif; background:#f5f5f7;}
.container {max-width:420px; margin:40px auto; background:#fff; padding:24px; border-radius:12px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
h1 {text-align:center;}
label {display:block; margin-top:10px;}
input[type=text],input[type=password]{width:100%; padding:8px; border:1px solid #ccc; border-radius:6px;}
button{width:100%; margin-top:16px; padding:10px; background:#007bff; color:#fff; border:none; border-radius:6px; font-weight:bold; cursor:pointer;}
.error{background:#ffecec; color:#c00; padding:10px; border-radius:6px;}
.success{background:#e0ffe0; color:#060; padding:10px; border-radius:6px;}
a{text-decoration:none; color:#007bff;}
</style>
</head>
<body>
<div class="container">
<h1>新規登録</h1>

<?php if ($done): ?>
  <div class="success">アカウントを作成しました。<br><a href="login.php">ログインページへ</a></div>
<?php else: ?>
  <?php if ($errors): ?>
    <div class="error">
      <?php foreach($errors as $e){ echo "・".h($e)."<br>"; } ?>
    </div>
  <?php endif; ?>

  <form method="post">
    <input type="hidden" name="token" value="<?php echo h($_SESSION['token']); ?>">
    <label>ユーザー名</label>
    <input type="text" name="username" value="<?php echo h($username); ?>" required>
    <label>パスワード</label>
    <input type="password" name="password" required>
    <label>パスワード（確認）</label>
    <input type="password" name="password_confirm" required>
    <button type="submit">登録</button>
  </form>
  <p style="text-align:center; margin-top:10px;">すでにアカウントをお持ちですか？ <a href="login.php">ログイン</a></p>
<?php endif; ?>
</div>
</body>
</html>
