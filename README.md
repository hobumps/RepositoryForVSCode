<?php
session_start();

// ログインチェック
if (empty($_SESSION['logged_in'])) {
    header('Location: login.php');
    exit;
}

// --- DB接続 ---
$dsn = 'mysql:host=localhost;dbname=tb270594db;charset=utf8mb4';
$user = 'tb-270594';
$password = 'w6fETMAwuw';
try {
    $pdo = new PDO($dsn, $user, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
} catch (PDOException $e) {
    exit('DB接続エラー: ' . $e->getMessage());
}

function h($s) { return htmlspecialchars($s, ENT_QUOTES, 'UTF-8'); }

// 自分の情報
$user_id = $_SESSION['user_id'];
$username = $_SESSION['username'];

// --- 受信者一覧の取得（自分以外のユーザー） ---
$stmt = $pdo->prepare('SELECT id, username FROM users WHERE id != ? ORDER BY id');
$stmt->execute([$user_id]);
$users = $stmt->fetchAll();

// --- メッセージ送信処理 ---
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['receiver_id'], $_POST['content'])) {
    $receiver_id = (int)$_POST['receiver_id'];
    $content = trim($_POST['content']);
    if ($content !== '') {
        $stmt = $pdo->prepare('INSERT INTO messages (sender_id, receiver_id, content) VALUES (?, ?, ?)');
        $stmt->execute([$user_id, $receiver_id, $content]);
    }
    header("Location: index.php?chat_with={$receiver_id}");
    exit;
}

// --- チャット相手の指定 ---
$chat_with = isset($_GET['chat_with']) ? (int)$_GET['chat_with'] : null;
$chat_user = null;
$messages = [];

if ($chat_with) {
    // 相手情報
    $stmt = $pdo->prepare('SELECT username FROM users WHERE id = ?');
    $stmt->execute([$chat_with]);
    $chat_user = $stmt->fetchColumn();

    if ($chat_user) {
        // メッセージ履歴（両方向）
        $stmt = $pdo->prepare('
            SELECT m.*, u.username AS sender_name
            FROM messages AS m
            JOIN users AS u ON m.sender_id = u.id
            WHERE (m.sender_id = ? AND m.receiver_id = ?)
               OR (m.sender_id = ? AND m.receiver_id = ?)
            ORDER BY m.sent_at ASC
        ');
        $stmt->execute([$user_id, $chat_with, $chat_with, $user_id]);
        $messages = $stmt->fetchAll();
    }
}
?>
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>チャットルーム</title>
<style>
body { font-family: sans-serif; display: flex; margin: 0; height: 100vh; }
.sidebar {
  width: 220px; background: #f4f4f4; padding: 15px; border-right: 1px solid #ccc;
}
.chat-area { flex: 1; display: flex; flex-direction: column; }
header {
  background: #0078d7; color: #fff; padding: 10px;
  display: flex; justify-content: space-between; align-items: center;
}
.messages {
  flex: 1; overflow-y: auto; padding: 15px; background: #fafafa;
}
.message {
  margin-bottom: 12px; padding: 10px 14px; border-radius: 10px;
  max-width: 60%; word-wrap: break-word;
}
.mine { background: #d1ecf1; align-self: flex-end; }
.theirs { background: #e2e3e5; align-self: flex-start; }
form {
  display: flex; padding: 10px; border-top: 1px solid #ccc;
}
textarea {
  flex: 1; resize: none; padding: 8px;
}
button {
  padding: 8px 14px; margin-left: 8px;
}
</style>
</head>
<body>
<div class="sidebar">
  <h3>こんにちは、<?php echo h($username); ?> さん</h3>
  <p><a href="logout.php">ログアウト</a></p>
  <hr>
  <h4>ユーザー一覧</h4>
  <ul>
    <?php foreach ($users as $u): ?>
      <li><a href="?chat_with=<?php echo $u['id']; ?>"><?php echo h($u['username']); ?></a></li>
    <?php endforeach; ?>
  </ul>
</div>

<div class="chat-area">
  <header>
    <span>
      <?php if ($chat_user): ?>
        <?php echo h($chat_user); ?> さんとのチャット
      <?php else: ?>
        チャット相手を選択してください
      <?php endif; ?>
    </span>
  </header>

  <div class="messages">
    <?php foreach ($messages as $m): ?>
      <div class="message <?php echo $m['sender_id'] == $user_id ? 'mine' : 'theirs'; ?>">
        <strong><?php echo h($m['sender_name']); ?>:</strong><br>
        <?php echo nl2br(h($m['content'])); ?><br>
        <small><?php echo h($m['sent_at']); ?></small>
      </div>
    <?php endforeach; ?>
  </div>

  <?php if ($chat_user): ?>
  <form method="post" action="">
    <input type="hidden" name="receiver_id" value="<?php echo $chat_with; ?>">
    <textarea name="content" rows="2" placeholder="メッセージを入力..." required></textarea>
    <button type="submit">送信</button>
  </form>
  <?php endif; ?>
</div>
</body>
</html>
